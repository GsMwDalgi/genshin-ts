# GIA vs GIL: Key Differences

## Shared Properties

| Property         | GIA         | GIL         |
|------------------|-------------|-------------|
| Header size      | 20 bytes    | 20 bytes    |
| Footer size      | 4 bytes     | 4 bytes     |
| head_tag         | 0x0326      | 0x0326      |
| tail_tag         | 0x0679      | 0x0679      |
| Byte order       | Big-endian  | Big-endian  |
| Content encoding | Protobuf    | Protobuf    |

## Key Differences

| Aspect             | GIA                        | GIL                                      |
|--------------------|----------------------------|------------------------------------------|
| **Purpose**        | Single node graph export   | Full map save (many graphs + metadata)   |
| **file_type**      | 3 (always)                 | Varies                                   |
| **schema_version** | 1 (always)                 | 1 (observed)                             |
| **Proto message**  | `Root` (gia.proto)         | Larger undocumented container             |
| **NodeGraph count**| Exactly 1                  | Many (one per graph in the map)          |
| **Folder index**   | None                       | Present at path 6.1.*                    |
| **Custom prefabs** | None                       | Present at path 4.1.*                    |
| **Signal defs**    | None (uses placeholders)   | Contains real signal CompositeDef entries |
| **File location**  | Build output (dist/)       | Game save folder (BeyondLocal)           |
| **Typical size**   | Small (single graph)       | Large (entire map)                       |

## GIA Proto Structure

```
Root
  .graph (GraphUnit)
    .id
    .name
    .which
    .graph (NodeGraphWrapper)      <- the single graph
      .inner
        .graph (NodeGraph)         <- target data
  .accessories[]
  .filePath
  .modeFlag?
  .gameVersion
```

## GIL Proto Structure (inferred from binary parsing)

```
(Unknown root)
  field 4: Custom prefab container
    4.1[]: Prefab entries
      4.1.6.11.1: prefab name
  field 6: Folder system
    6.1[]: Folder entries
      6.1.1: folder ID
      6.1.2: folder metadata
        6.1.2.4[]: meta lists with entries
      6.1.3: folder content
        6.1.3.1: folder name
        6.1.3.5[]: content entries (typeValue, id)
  field 10: Graph container
    10.1: inner wrapper
      10.1.1[]: NodeGraph blobs (same as gia.proto NodeGraph)
```

The GIL proto schema is not fully reverse-engineered. The injector only needs to understand:
1. Where NodeGraph blobs live (10.1.1) for replacement
2. The folder index (6.1) for graph type resolution
3. Signal definitions scattered in composite defs

## Relationship During Injection

```
GIA file                    GIL file
─────────                   ─────────
Root.graph.graph            10.1.1[N]
  .inner.graph   ────────>  (replaced NodeGraph blob)
     (NodeGraph)            matching by ID
```

The injector extracts the NodeGraph from the GIA's wrapper structure, then binary-patches it into the matching slot in the GIL's 10.1.1 blob array.
