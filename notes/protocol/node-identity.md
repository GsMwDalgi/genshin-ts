# Node Identity

## Node IDs: genericId vs concreteId [INFERRED]

Each node has two IDs encoded as `NodeProperty`:

- **genericId**: 노드의 템플릿 종류 (e.g., send_signal = 300000). Always `SystemDefined` class, `SysCall` kind for standard nodes.
- **concreteId**: 제네릭/리플렉티브 노드의 타입 특수화 시 사용. 일반 노드는 `concreteId` 없음.

```protobuf
NodeProperty {
  class = 10001  (SystemDefined)
  type  = 20000  (Server) or 20001 (Filter) or 20002 (Skill) or 20007 (CreationStatusNode)
  kind  = 22000  (SysCall) or 22001 (SysGraph)
  nodeId = <NODE_ID enum value>
}
```

### Reflective Node Handling [CONFIRMED]

For reflective (generic) nodes, `concreteId` determines the concrete type instantiation. Pin values use `ConcreteBase` (10000) class with `indexOfConcrete` to map to specific type positions.

## Special Node IDs [CONFIRMED]

| Node Type           | ID     | Usage          |
|---------------------|--------|----------------|
| send_signal         | 300000 | 신호 전송       |
| monitor_signal      | 300001 | 신호 수신       |
| assemble_structure  | 300002 | 구조체 조립     |
| split_structure     | 300003 | 구조체 분해     |
| modify_structure    | 300004 | 구조체 수정     |

## Injector ID Remapping [CONFIRMED]

- 컴파일 시 IR JSON의 노드 ID는 임시값 (2, 3, 4, ...)
- 특수 노드는 고정 ID 사용 (300000, 300001 등)
- 인젝터가 맵에 존재하는 실제 노드 ID로 패치

## General Node Categories [INFERRED]

- 이벤트 노드 (`when_entity_is_created` 등): 에디터의 사전 정의 시스템 노드
- 실행 노드 (`print_string`, `set_local_variable` 등): SysCall(22000) 범주
- 사용자 정의 노드: NodeGraph(21001) 범주

## Patched Node IDs in GIL [CONFIRMED]

인젝터가 GIL에 삽입할 때 특수 노드 ID(300000 등)를 맵 고유 ID로 패치한다.
GIL에서 디코딩하면 원래의 300000이 아닌 패치된 값이 나온다:

| Original (IR/GIA) | After GIL Patch | Role                              |
|--------------------|-----------------|-----------------------------------|
| 300000             | 1610612737      | send_signal                       |
| 300001             | 1610612738      | monitor_signal (onSignal)         |
| (observed)         | 1610612744      | monitor_signal (별도 인스턴스)     |

**주의**: 패치된 ID는 맵마다 다를 수 있다. NODE_ID 테이블에 존재하지 않으므로
역방향 조회 시 `node_<id>` 폴백 이름으로 표시된다.

### Signal Node Reverse Identification [CONFIRMED]

NODE_ID에 없는 nodeId를 가진 노드 중, `kind: "SysGraph"` 속성을 가진 노드가 시그널 노드이다.
시그널 이름은 `ClientExecNode` kind 핀의 `value.bString.val`에 저장되어 있다.

**주의**: protobuf enum은 런타임에서 정수값이다. `JSON.stringify()`/`toJSON()` 시에만
문자열로 변환된다. 반드시 양쪽을 모두 비교해야 한다:

| Field                  | Integer | String (toJSON) |
|------------------------|---------|-----------------|
| `genericId.kind` (SysGraph) | 22001 | `"SysGraph"` |
| `pin.i1.kind` (ClientExecNode) | 5  | `"ClientExecNode"` |

```ts
// 시그널 노드 판별 -- 정수와 문자열 양쪽 비교 필수
const SYS_GRAPH_KIND = 22001
node.genericId?.kind === SYS_GRAPH_KIND || node.genericId?.kind === 'SysGraph'

// 시그널 이름 추출
for (const pin of node.pins) {
  if (pin.i1?.kind === 5 || pin.i1?.kind === 'ClientExecNode') {
    return pin.value?.bString?.val  // 'NextEffect', '_RefLoop' 등
  }
}
```

## Sources

- `ir_to_gia_transform/mappings.ts` — SPECIAL_NODE_IDS
- `gia_gen/graph.ts` — Node.encode() 로직
- `node_data/helpers.ts` — get_index_of_concrete
- GIL 역방향 분석 (2026-03-29) — SampleForEffect 맵에서 protobuf 디코딩 확인
