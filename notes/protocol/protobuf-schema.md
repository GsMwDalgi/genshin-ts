# Protobuf Schema

Source: `src/thirdparty/.../protobuf/gia.proto` (version 1.5.0)

## Root Message [CONFIRMED]

```protobuf
message Root {
  GraphUnit graph = 1;                  // 메인 그래프 정보
  repeated GraphUnit accessories = 2;   // 데이터 구조, 입출력
  string filePath = 3;                  // "{UID}-{TIME}-{LEVEL_ID}-\{FILE_NAME}.gia"
  optional uint32 modeFlag = 4;         // classic=1, Beyond=생략
  string gameVersion = 5;               // 예: "6.3.0"
}
```

### modeFlag [CONFIRMED]

| Value   | Mode                    |
|---------|-------------------------|
| omitted | Beyond 모드 (기본)       |
| 1       | Classic 모드             |

### filePath Format

`{uid}-{timestamp_seconds}-{file_id}-\{graph_name_normalized}`

- `uid`: 9-digit random number (prefix "201")
- `timestamp`: Unix seconds at generation time
- `file_id`: usually `graph_id + counter`
- Graph name is lowercased, non-alphanumeric chars replaced with `_`

## GraphUnit (Top-level Container) [CONFIRMED]

Each GraphUnit wraps one node graph. The `which` field determines the graph category.

### GraphUnit.Which

| Which | Name                    | Description                    |
|-------|-------------------------|--------------------------------|
| 0     | Unknown                 |                                |
| 9     | EntityNode              | Standard entity graph          |
| 10    | BooleanFilter           | Boolean filter graph           |
| 11    | Skills                  | Skills graph                   |
| 12    | CompositeGraph          | Composite (user-defined subgraph) |
| 22    | StatusNode              | Status node graph              |
| 23    | ClassNode               | Class node graph               |
| 29    | StructureDefinition     | Structure type definition      |
| 46    | ItemNode                | Item node graph                |
| 47    | IntegerFilter           | Integer filter graph           |
| 51    | CreationStatusDecision  |                                |
| 52    | CreationSkill           |                                |
| 53    | CreationStatus          |                                |

### GraphUnit.Id [CONFIRMED]

```protobuf
message Id {
  enum Class { Unknown1=0; Node=1; Basic=5; AffiliatedNode=23; }
  enum Type  { ServerGraph=0; ClientGraph=3; StructureDefinition=15; }
  Class class = 2;
  Type type = 3;
  int32 id = 4;    // Node Graph Id
}
```

### GraphUnit.graphType (oneof) [CONFIRMED]

| Field    | Wrapper               | Contents     | Usage                           |
|----------|-----------------------|--------------|---------------------------------|
| field 13 | NodeGraphWrapper      | NodeGraph    | Most common (standard graphs)   |
| field 14 | CompositeDefWrapper   | CompositeDef | Composite graphs                |
| field 22 | StructureDefWrapper   | StructureDef | Structure type definitions      |

## NodeGraph (Core Graph Object) [CONFIRMED]

```protobuf
message NodeGraph {
  Id id = 1;
  string name = 2;
  repeated GraphNode nodes = 3;
  repeated CompositePin compositePins = 4;
  repeated Comments comments = 5;
  repeated GraphVariable graphValues = 6;
  repeated GraphAffiliation affiliations = 7;
  optional int32 entrySlotIndex = 100;
  optional float evaluationInterval = 101;
}
```

### NodeGraph.Id [CONFIRMED]

```protobuf
message Id {
  enum Class { Unknown1=0; UserDefined=10000; SystemDefined=10001; }
  enum Type  { Unknown2=0; BasicNode=20000; BooleanFilter=20001; Skills=20002;
               StatusNode=20003; ClassNode=20004; ItemNode=20005;
               IntegerFilter=20006; CreationStatusDecision=20007;
               CreationSkill=20008; CreationStatus=20009; }
  enum Kind  { Unknown3=0; NodeGraph=21001; CompositeGraph=21002;
               SysCall=22000; SysGraph=22001; }
  Class class = 1;
  Type type = 2;
  Kind kind = 3;
  int64 id = 5;    // Unique graph ID (typically >= 1,000,000,000)
}
```

## GraphNode (Individual Node) [CONFIRMED]

```protobuf
message GraphNode {
  int32 nodeIndex = 1;                        // 그래프 내 고유 인덱스
  NodeProperty genericId = 2;                 // 노드 템플릿/제네릭 ID
  optional NodeProperty concreteId = 3;       // 특수화된 구체 ID (리플렉티브 노드 전용)
  repeated NodePin pins = 4;                  // 핀 목록
  float x = 5;                               // 캔버스 X 좌표
  float y = 6;                               // 캔버스 Y 좌표
  optional Comments comments = 7;             // 코멘트
  optional NodePin.Index contextDeclaration = 8;   // client graph entry context
  optional int32 signalVersion = 9;                // signal/listen declaration version
  repeated GraphAffiliation.Info usingStruct = 10; // 구조체 참조
  optional StatusNodeExtension statusNodeExtension = 11;
}
```

### nodeIndex [CONFIRMED]

- 그래프 내에서 유일한 정수 ID
- IR JSON의 `node.id`와 대응
- 인젝터가 실제 ID로 재매핑 (예: 300000 -> 실제 send_signal 노드 ID)

### Position Encoding [CONFIRMED]

Positions are stored as `logical_pos * scale + random_noise(0~10)`:
- x: scale = 300
- y: scale = 200

## NodeProperty [INFERRED]

```protobuf
message NodeProperty {
  NodeGraph_Id_Kind kind = 1;    // 노드 분류
  NodeGraph_Id_Class class = 2;  // 사용자/시스템 정의
  NodeGraph_Id_Type type = 3;    // 그래프 모드
  int32 id = 4;                  // 노드 타입 고유 ID
}
```

### NodeGraph_Id_Kind [CONFIRMED]

| Value | Name           | Usage                             |
|-------|----------------|-----------------------------------|
| 21001 | NodeGraph      | 사용자 정의 그래프                  |
| 21002 | CompositeGraph | 사용자 정의 복합 그래프             |
| 22000 | SysCall        | 시스템 콜 (사전 정의)               |
| 22001 | SysGraph       | 사용자 struct에서 생성 / 시그널 노드 |

### NodeGraph_Id_Class [CONFIRMED]

| Value | Name           |
|-------|----------------|
| 10000 | UserDefined    |
| 10001 | SystemDefined  |

### NodeGraph_Id_Type (Graph Mode) [CONFIRMED]

| Value | Name           | Usage                |
|-------|----------------|----------------------|
| 20000 | BasicNode      | 기본 서버 노드그래프  |
| 20001 | BooleanFilter  | 부울 필터 노드        |
| 20002 | Skills         | 스킬 노드            |
| 20003 | StatusNode     | 상태 노드그래프       |
| 20004 | ClassNode      | 클래스 노드그래프     |
| 20005 | ItemNode       | 아이템 노드그래프     |
| 20006 | IntegerFilter  | 정수 필터 노드        |

## Sources

- `gia.proto` / `gia.proto.ts` — protobuf 타입 정의
- `gia_gen/graph.ts` — Graph/Node encode 로직
