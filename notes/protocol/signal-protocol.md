# Signal Node Protocol

## send_signal (ID: 300000) [CONFIRMED]

### Pin Layout

```
+------------------------------+
|       send_signal            |
+------------------------------+
| InFlow[0]     <- 실행 흐름   |
| ClientExec[0] <- 신호 이름   |  <- kind=5, 문자열 리터럴
| InParam[0]    <- 첫번째 인자 |  <- kind=3, 리터럴 또는 conn
| InParam[1]    <- 두번째 인자 |
| ...                          |
| OutFlow[0]    -> 실행 흐름   |
+------------------------------+
```

### IR -> GIA Conversion Rules

| IR args index       | GIA Pin       | Kind |
|---------------------|---------------|------|
| args[0] (signal name) | ClientExec[0] | 5  |
| args[1] (first arg)   | InParam[0]    | 3  |
| args[2] (second arg)  | InParam[1]    | 3  |
| args[N]              | InParam[N-1]  | 3  |

### Literal Arg Encoding [CONFIRMED]

`setLiteralArgValue(giaNode, pinIndex, pinIndex, nodeType, argType, argValue)`
- InParam 핀에 타입 설정 + 리터럴 값 직접 기록
- pinIndex는 0-based (InParam 네임스페이스)

### Connection Arg Encoding [CONFIRMED]

`ensureInputPinWithType(giaNode, pinIndex, argType)`
- 값 없이 타입 정보만 가진 빈 InParam 핀 생성
- `Pin.encode()`가 `connects`에서 `to_index === pin.index` 매칭으로 데이터 연결 인코딩
- **핀이 존재하지 않으면 연결이 누락됨** (초기 구현 버그의 원인)

### Index Remapping [CONFIRMED]

```
remapInputIndexForHiddenPin('send_signal', irToIndex)
-> return irToIndex - 1
```

이유: IR에서 args[0]이 ClientExec이므로 첫 데이터 인자는 args[1] (toIndex=1).
GIA에서 InParam은 0-based이므로 toIndex-1 = InParam[0].

---

## monitor_signal (ID: 300001) [CONFIRMED]

### Pin Layout

```
+------------------------------+
|     monitor_signal           |
+------------------------------+
| ClientExec[0] <- 신호 이름   |
| OutFlow[0]    -> 실행 흐름   |
| OutParam[0]   -> eventSourceEntity   |  <- 고정
| OutParam[1]   -> eventSourceGuid     |  <- 고정
| OutParam[2]   -> signalSourceEntity  |  <- 고정
| OutParam[3]   -> 첫번째 커스텀 인자  |  <- 동적
| OutParam[4]   -> 둘째 커스텀 인자    |  <- 동적
| ...                          |
+------------------------------+
```

### Fixed Output Pins [CONFIRMED]

| OutParam Index | Name                | Type   | Source           |
|----------------|---------------------|--------|------------------|
| 0              | eventSourceEntity   | entity | events.js 정의    |
| 1              | eventSourceGuid     | guid   | events.js 정의    |
| 2              | signalSourceEntity  | entity | events.js 정의    |

### Custom Arg Output Pins [CONFIRMED]

- OutParam[3]부터 시작
- `signalArgs` 배열 순서대로 추가
- `markPin(eventNode, argDef.name, eventArgs.length)` 으로 핀 등록
- `eventObj[argDef.name]` 으로 핸들러에서 접근 가능

---

## Signal Name Extraction (Injector) [CONFIRMED]

인젝터가 protobuf-decoded 노드에서 신호 이름을 추출할 때:

```javascript
// 1순위: ClientExec(kind=5) 핀에서 문자열 추출
for (pin of pins) {
  if ((pin.i1?.kind ?? pin.i2?.kind) === 5) {
    return extractStringFromVarBase(pin.value)
  }
}
// 2순위: 아무 핀에서 첫 문자열 추출 (폴백)
for (pin of pins) {
  return extractStringFromVarBase(pin.value)
}
```

**주의**: InParam 핀이 ClientExec 앞에 올 수 있음 (ensureInputPin의 splice 때문).
인자에 문자열 리터럴이 있으면 신호 이름으로 오인될 수 있어 kind=5 우선 검색 필요.

### Injector Pin Access Pattern [CONFIRMED]

```javascript
// 인젝터는 protobuf-decoded 객체를 다루므로:
pin.i1.kind  // Pin 클래스의 pin.kind와 다름
pin.i2.kind  // i1과 동일한 값
```

---

## signalParams Metadata [CONFIRMED]

IR JSON과 GIA 모두에 보존되는 인자 스키마:

```json
"signalParams": [
  { "name": "argInt", "type": "int" },
  { "name": "argStr", "type": "str" }
]
```

- send_signal / monitor_signal 모두 동일한 signalParams 보유
- IR 빌더에서 `buildSignalNode`가 기본 빌더를 확장하여 보존
- 인자 없는 신호는 signalParams 필드 자체가 없음 (하위 호환)

## Sources

- `ir_to_gia_transform/index.js` — applySpecialArgs, remapInputIndexForHiddenPin
- `ir_to_gia_transform/pins.js` — setLiteralArgValue, ensureInputPinWithType
- `runtime/ir_builder.js` — buildSignalNode
- `injector/signal_nodes.js` — extractSignalNameFromNode
- signal-args-implementation.md
