# GIL Binary Format

## Overview

GIL (Genshin Impact Level) files are the map save files for Beyond Mode. Each `.gil` file represents one saved map containing scene objects, components, node graphs, and other map data.

## File Location

```
%LOCALAPPDATA%/../LocalLow/miHoYo/{Game}/BeyondLocal/{playerId}/Beyond_Local_Save_Level/{mapId}.gil
```

Game folder names:
- China: `原神`
- Global: `Genshin Impact`

Resolution logic (from `gil_paths.ts`):
- Auto-detection: if only one region folder exists, use it; if only one numeric dir, use it

## Binary Envelope [CONFIRMED]

GIL files share the **same binary envelope** as GIA files:

| Offset | Size | Field            | Value / Description           |
|--------|------|------------------|-------------------------------|
| 0x00   | 4B   | `left_size`      | `file_size - 4` (big-endian)  |
| 0x04   | 4B   | `schema_version` | `1` (observed)                |
| 0x08   | 4B   | `head_tag`       | `0x0326` (same as GIA)        |
| 0x0C   | 4B   | `file_type`      | `2` (GIL, vs GIA's `3`)      |
| 0x10   | 4B   | `proto_size`     | `file_size - 24`              |
| 0x14   | ...  | protobuf payload | (variable length)             |
| EOF-4  | 4B   | `tail_tag`       | `0x0679` (same as GIA)        |

Header: 20 bytes. Footer: 4 bytes. All values big-endian.

## Protobuf Root Structure [INFERRED]

GIL은 GIA의 `Root` 메시지와 **다른 스키마**를 사용한다. `gia.proto`의 `Root`로 디코딩하면 wire type 오류 발생.

### Top-Level Fields (raw decode, 1073741960-temp.gil) [INFERRED]

| Field | Estimated Role       | Size (example) | Notes                            |
|-------|---------------------|----------------|----------------------------------|
| 2     | 맵 메타데이터?       | 15B            |                                  |
| 4     | Custom prefabs       | 1208B          | 커스텀 프리팹 컨테이너             |
| 5     | **Scene Objects**    | 4363B          | repeated, 씬에 배치된 오브젝트 목록 |
| 6     | **Folder system**    | 1162B          | 32 children, 그래프 분류 인덱스    |
| 9     | ?                    | 1924B          |                                  |
| 10    | **NodeGraph Data**   | 5593B          | GIA Root와 유사한 노드그래프 데이터 |
| 11    | ?                    | 364B           |                                  |
| 14    | ?                    | 78B            |                                  |
| 15    | ?                    | 5740B          |                                  |
| 기타   | 빈 필드 다수         | 0B             | 3,7,8,12,16-21,27,30-33,37,44 등 |

## Folder Index (path prefix 6.1) [CONFIRMED]

The folder system organizes node graphs into categories:

```
6.1           FolderEntry (repeated)
  6.1.1       FolderId (varint)
  6.1.3       ContentField
    6.1.3.1     name (string, folder name)
    6.1.3.5     entries (repeated)
      .1          typeValue — category value
      .2          id — NodeGraph id reference
  6.1.2.4     MetaList
    6.1.2.4.1   name (string)
    6.1.2.4.5   entries (repeated)
```

### Category Value to Graph Type Mapping [CONFIRMED]

| typeValue | Graph Type (NodeGraph.Id.Type) | Name           |
|-----------|-------------------------------|----------------|
| 800       | 20000                         | Entity (basic) |
| 2300      | 20003                         | Status         |
| 2400      | 20004                         | Class          |
| 4300      | 20005                         | Item           |

## NodeGraph Blobs (path prefix 10.1.1) [CONFIRMED]

Each NodeGraph is stored as a nested protobuf blob:

```
10            Root container
  10.1        Inner wrapper
    10.1.1    NodeGraph blob (self-contained protobuf)
```

These blobs use the same `NodeGraph` protobuf message as GIA files (defined in `gia.proto`). Each blob can be independently decoded using `protobufjs`.

### NodeGraph Identification

The fast scanner in `node_graph.ts` reads IDs without full protobuf decode:
1. Verify first key varint is 10 (field 1, wire type 2 = embedded message)
2. Read the length-delimited Id sub-message
3. Extract `type` (field 2) and `id` (field 5) as varints

## Custom Prefab Extraction [CONFIRMED]

GIL files contain custom prefab definitions at protobuf path:

```
4.1           Custom prefab entry (depth=2, p0=4, p1=1)
  4.1.6.11.1  Prefab name (depth=5, field=1) — UTF-8 string
  Entry fields:
    field 1   customId (varint)
    field 2   basePrefabId (varint)
```

Extracted to generate TypeScript source files with custom prefab constants.

## Sources

- GIL 바이너리 raw protobuf decode
- `genshin-ts/src/injector/node_graph.ts` — NodeGraph 스캐너
- `genshin-ts/src/injector/gil_paths.ts` — 파일 경로 해석
- `genshin-ts/src/injector/gil_resources.ts` — 커스텀 프리팹 추출
