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
| **file_type**      | 3                          | 2                                        |
| **schema_version** | 1 (always)                 | 1 (observed)                             |
| **Proto message**  | `Root` (gia.proto)         | Larger undocumented container            |
| **NodeGraph count**| Exactly 1                  | Many (one per graph in the map)          |
| **Folder index**   | None                       | Present at path 6.1.*                    |
| **Custom prefabs** | None                       | Present at path 4.1.*                    |
| **Signal defs**    | None (uses placeholders)   | Contains real signal CompositeDef entries |
| **File location**  | Build output (dist/)       | Game save folder (BeyondLocal)           |
| **Typical size**   | Small (single graph)       | Large (entire map)                       |

### 추가 차이점 (from binary analysis) [CONFIRMED]

- GIL은 에디터가 디스크에 저장하는 맵 파일 포맷, GIA는 인젝션용 노드그래프 포맷
- GIL은 GIA의 `Root` 메시지와 다른 protobuf 스키마를 사용 (GIA Root로 디코딩 시 wire type 오류)
- GIL은 씬 오브젝트, 컴포넌트, 노드그래프 등 맵 전체 데이터를 포함

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
  field 5: Scene Objects (repeated)
    5.f1[]: Scene object entries
      f1: object_id
      f7: Components (repeated)
  field 6: Folder system
    6.1[]: Folder entries
      6.1.1: folder ID
      6.1.3: folder content (name, entries with typeValue/id)
  field 10: Graph container
    10.1: inner wrapper
      10.1.1[]: NodeGraph blobs (same as gia.proto NodeGraph)
```

GIL의 protobuf 스키마는 완전히 역공학되지 않았다. 인젝터가 필요한 부분만 파악:
1. NodeGraph blob 위치 (10.1.1) — replacement 대상
2. Folder index (6.1) — 그래프 타입 해석
3. Signal definitions in composite defs — 시그널 노드 ID 해석

## Relationship During Injection

```
GIA file                    GIL file
---------                   ---------
Root.graph.graph            10.1.1[N]
  .inner.graph   -------->  (replaced NodeGraph blob)
     (NodeGraph)            matching by ID
```

인젝터는 GIA의 wrapper 구조에서 NodeGraph를 추출한 후, GIL의 10.1.1 blob 배열에서 ID가 일치하는 슬롯에 바이너리 패치한다.
