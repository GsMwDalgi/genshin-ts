# Pin System

## Pin Kinds (NodePin_Index_Kind) [CONFIRMED]

| Kind | Name           | Direction | Usage                                    |
|------|----------------|-----------|------------------------------------------|
| 1    | InFlow         | Input     | 실행 흐름 입력 (이전 노드에서 연결)         |
| 2    | OutFlow        | Output    | 실행 흐름 출력 (다음 노드로 연결)           |
| 3    | InParam        | Input     | 데이터 입력 파라미터                        |
| 4    | OutParam       | Output    | 데이터 출력 파라미터                        |
| 5    | ClientExecNode | Input     | 클라이언트 실행 노드 (signal 전용)          |
| 6    | ClientSignal   | Input     | Client signal/topic metadata              |

Source: `gia.proto` (verified)

## Index Namespaces [CONFIRMED]

**핵심 규칙: 각 kind는 독립적인 인덱스 공간을 가짐.**

```
InFlow[0], InFlow[1], ...     <- kind=1 namespace
OutFlow[0], OutFlow[1], ...   <- kind=2 namespace
InParam[0], InParam[1], ...   <- kind=3 namespace
OutParam[0], OutParam[1], ... <- kind=4 namespace
ClientExec[0]                 <- kind=5 namespace
ClientSignal[0]               <- kind=6 namespace
```

- InParam[0]과 ClientExec[0]은 충돌하지 않음
- Signal 노드에서 신호 이름은 ClientExec[0], 인자는 InParam[0]부터 시작
- 이 사실은 signal args 바이너리 비교에서 확정됨

## Pin Protobuf Structure [CONFIRMED]

```protobuf
message NodePin {
  Index i1 = 1;                           // primary index info
  Index i2 = 2;                           // duplicate of i1
  VarBase value = 3;                      // literal value (InParam only)
  int32 type = 4;                         // VarType (server) or ClientVarType (client)
  repeated NodeConnection connects = 5;   // connection list
  optional Index clientExecNode = 6;      // kind=5 전용 metadata
  optional int32 compositePinIndex = 7;   // composite node 호출 시 전용
}
```

### NodePin.Index [CONFIRMED]

```protobuf
message NodePin.Index {
  enum Kind { Unknown=0; InFlow=1; OutFlow=2; InParam=3; OutParam=4;
              ClientExecNode=5; ClientSignal=6; }
  Kind kind = 1;
  int32 index = 2;   // kind 내 인덱스

  message Id { int64 id = 1; }
  oneof clientExecNode {   // only for client nodes
    Id nodeId = 100;
  }
}
```

## ClientExecNode Field (field 6) [CONFIRMED]

kind=5 핀에는 추가 필드가 붙음:

```protobuf
message ClientExecNode {
  int32 kind = 1;    // 항상 5
  int32 index = 2;   // 항상 1 (고정)
}
```

## Pin Value Encoding (VarBase) [CONFIRMED]

Non-reflective (simple) pins:
```
VarBase {
  class = <matching VarBase.Class>
  alreadySetVal = true
  bInt/bFloat/bString/bEnum/bVector = { val: <value> }
}
```

Reflective (concrete) pins:
```
VarBase {
  class = ConcreteBase (10000)
  alreadySetVal = true
  bConcreteValue = {
    indexOfConcrete = <pin-specific concrete index>
    value = <inner VarBase with actual value>
    structs? = <ComplexValueStruct for struct/map/array types>
  }
}
```

### Index of Concrete [CONFIRMED]

For reflective nodes, each pin position maps to a concrete type index:
```typescript
get_index_of_concrete(generic_id, is_input, pin_index, node_type)
```

## Pin Array Ordering [CONFIRMED]

GIA 바이너리에서 `pins[]` 배열 내 핀의 물리적 순서:

1. InParam 핀이 먼저 (index 0부터)
2. ClientExec 핀이 그 뒤에 밀려남

`ensureInputPin()`이 InParam을 splice로 삽입하므로, 기존 핀(ClientExec 포함)이 오른쪽으로 밀림.

**주의**: 이로 인해 `signal_nodes.js`에서 신호 이름 추출 시 kind=5 우선 검색이 필요. InParam 핀이 ClientExec 앞에 올 수 있으므로, 인자에 문자열 리터럴이 있으면 신호 이름으로 오인될 수 있다.

## Pin.encode() Behavior [CONFIRMED]

1. 타입이 결정되지 않은 핀은 `null` 반환 (인코딩 제외)
2. InParam 핀: value가 있으면 리터럴, 없으면 conn 전용
3. **conn 매칭**: `connects.find(c => c.to_index === this.index)` — 핀이 존재해야 연결이 인코딩됨
4. 핀이 없으면 연결이 누락됨 (signal args conn 버그의 근본 원인)

## Protobuf Enum String/Integer Issue [CONFIRMED]

GIA 인코딩 시에는 kind가 정수(1-6)이지만, `loadGiaProto().nodeGraphMessage.decode()`로 디코딩하면
**protobuf enum이 문자열로 해석**된다:

| Integer | Decoded String     |
|---------|--------------------|
| 1       | `"InFlow"`         |
| 2       | `"OutFlow"`        |
| 3       | `"InParam"`        |
| 4       | `"OutParam"`       |
| 5       | `"ClientExecNode"` |
| 6       | `"ClientSignal"`   |

따라서 디코딩된 NodeGraph 객체를 다룰 때는 **정수와 문자열 양쪽을 모두 처리**해야 한다:

```ts
// 올바른 비교
if (pin.i1?.kind === 'OutFlow' || pin.i1?.kind === 2) { ... }
```

이것은 `injector/reader.ts`에서 이벤트 엔트리 감지 시 발견된 버그의 원인이었다.

## Sources

- `gia.proto` — NodePin_Index_Kind 열거형, NodePin 메시지 정의
- `gia_gen/graph.ts` — Pin 클래스, encode() 메서드
- `gia_gen/basic.ts` — ensureInputPin, 핀 순서 로직
- `node_data/helpers.ts` — get_index_of_concrete
- signal-args-implementation.md — 버그 수정 이력
