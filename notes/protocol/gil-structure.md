# GIL File Structure

## Overview

GIL (Genshin Impact Level) files are the map save files for Beyond Mode. Each `.gil` file represents one saved map and is stored at:

```
%LOCALAPPDATA%/../LocalLow/miHoYo/{Game}/BeyondLocal/{playerId}/Beyond_Local_Save_Level/{mapId}.gil
```

Game folder names:
- China: `原神`
- Global: `Genshin Impact`

## Binary Envelope

GIL files share the **same binary envelope** as GIA files:

```
Offset  Size   Field                Value / Description
------  ----   -----                -------------------
0x00    4B     left_size            file_size - 4 (big-endian uint32)
0x04    4B     schema_version       uint32 (observed: 1)
0x08    4B     head_tag             0x0326 (magic number, same as GIA)
0x0C    4B     file_type            uint32 (may differ from GIA's 3)
0x10    4B     proto_size           file_size - 24
0x14    ...    protobuf payload     (variable length)
EOF-4   4B     tail_tag             0x0679 (magic number, same as GIA)
```

Header: 20 bytes. Footer: 4 bytes. All values big-endian.

### Key Difference from GIA
- GIA: `file_type = 3`, protobuf payload is a `Root` message (single graph)
- GIL: Contains multiple NodeGraphs embedded in a larger protobuf structure

## GIL Protobuf Structure

The GIL protobuf is NOT the same `Root` message as GIA. It is a larger container. Key structural paths found by the binary parser:

### Folder Index (path prefix 6.1)

The folder system organizes node graphs into categories:

```
6.1           FolderEntry (repeated) — top-level folder entries
  6.1.1       FolderId (varint field 1)
  6.1.3       ContentField — folder contents
    6.1.3.1     name (string, folder name)
    6.1.3.5     entries (repeated FolderEntry)
      .1          typeValue — category value (e.g. 800, 2300, 2400, 4300)
      .2          id — NodeGraph id reference
  6.1.2.4     MetaList — additional metadata lists
    6.1.2.4.1   name (string)
    6.1.2.4.5   entries (repeated FolderEntry)
```

### Category Value to Graph Type Mapping

| typeValue | Graph Type (NodeGraph.Id.Type) | Name          |
|-----------|-------------------------------|---------------|
| 800       | 20000                         | Entity (basic)|
| 2300      | 20003                         | Status        |
| 2400      | 20004                         | Class         |
| 4300      | 20005                         | Item          |

### NodeGraph Blobs (path prefix 10.1.1)

Each NodeGraph is stored as a nested protobuf blob at depth 3:

```
10            Root container
  10.1        Inner wrapper
    10.1.1    NodeGraph blob (self-contained protobuf)
```

These blobs use the same `NodeGraph` protobuf message as GIA files (defined in gia.proto). Each blob can be independently decoded using `protobufjs`.

### NodeGraph Identification

Within each 10.1.1 blob, the graph can be identified by reading:
```
NodeGraph.id.id   (field 1 -> field 5, varint) — unique graph ID
NodeGraph.id.type (field 1 -> field 2, varint) — graph type enum
```

The fast scanner in `node_graph.ts` reads these without full protobuf decode:
1. Verify first key varint is 10 (field 1, wire type 2 = embedded message)
2. Read the length-delimited Id sub-message
3. Extract `type` (field 2) and `id` (field 5) as varints

## GIL File Locations

Resolution logic (from `gil_paths.ts`):

```
BeyondLocal root:
  China:  {LocalLow}/miHoYo/原神/BeyondLocal/
  Global: {LocalLow}/miHoYo/Genshin Impact/BeyondLocal/

Full path:
  {BeyondLocal}/{playerId}/Beyond_Local_Save_Level/{mapId}.gil
```

- `playerId`: numeric directory name under BeyondLocal
- `mapId`: numeric, becomes the filename `{mapId}.gil`
- Auto-detection: if only one region folder exists, use it; if only one numeric dir, use it

## Custom Prefab Extraction from GIL

GIL files also contain custom prefab definitions. The extraction logic (from `gil_resources.ts`):

### Prefab Entry Structure (protobuf path)
```
4.1           Custom prefab entry (depth=2, p0=4, p1=1)
  4.1.6.11.1  Prefab name (depth=5, field=1) — UTF-8 string
  Entry fields:
    field 1   customId (varint) — unique custom prefab ID
    field 2   basePrefabId (varint) — ID of the base prefab template
```

These are extracted to generate TypeScript source files with custom prefab constants for use in genshin-ts projects.
