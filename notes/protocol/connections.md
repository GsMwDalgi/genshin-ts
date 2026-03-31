# Connection Encoding

## NodeConnection Structure [CONFIRMED]

```protobuf
message NodeConnection {
  int32 id = 1;                    // 상대 노드 인덱스 (target or source nodeIndex)
  NodePin.Index connect = 2;       // 상대 핀 정보
  NodePin.Index connect2 = 3;      // connect의 복사본 (중복)
}
```

**`connect`와 `connect2`는 동일한 값.** 용도 불명이나 항상 같은 값으로 설정됨.

## Connection Direction [CONFIRMED]

### Data Connections (InParam <- OutParam)

InParam 핀의 `connects[]`에 소스 노드 정보 기록:

```
Target InParam[i].connects = [{
  id: sourceNodeIndex,
  connect:  { kind: OutParam(4), index: sourceOutParamIndex },
  connect2: { kind: OutParam(4), index: sourceOutParamIndex }
}]
```

데이터 연결은 "이 핀은 fromNodeId의 OutParam[fromPinIndex]에서 데이터를 받음" 의미.

### Flow Connections (OutFlow -> InFlow)

OutFlow 핀의 `connects[]`에 대상 노드 정보 기록:

```
Source OutFlow[i].connects = [{
  id: targetNodeIndex,
  connect:  { kind: InFlow(1), index: targetInFlowIndex },
  connect2: { kind: InFlow(1), index: targetInFlowIndex }
}]
```

플로우 연결은 "이 핀은 toNodeId의 InFlow[toPinIndex]로 흐름을 보냄" 의미.

> **Note**: Secondary source (graph-encoding.md) had the connect.kind values swapped (data used InParam(3) and flow used OutFlow(2)). This is **incorrect**. The connect field references the *remote* pin: data connections reference the source OutParam(4), flow connections reference the target InFlow(1).

## Encoding Functions [CONFIRMED]

```typescript
// 데이터 연결: "이 핀은 fromNodeId의 OutParam[fromPinIndex]에서 데이터를 받음"
node_connect_from(fromNodeId, fromPinIndex)
  -> { id: fromNodeId, connect: {kind: 4, index: fromPinIndex} }

// 플로우 연결: "이 핀은 toNodeId의 InFlow[toPinIndex]로 흐름을 보냄"
node_connect_to(toNodeId, toPinIndex)
  -> { id: toNodeId, connect: {kind: 1, index: toPinIndex} }
```

## Pin.encode() Connection Matching [CONFIRMED]

```typescript
// InParam 핀에서 데이터 연결 매칭
connects.find(c => this.kind === 3 && c.to_index === this.index)
```

**핵심**: 핀이 존재해야 연결이 인코딩됨. 핀이 없으면 연결이 누락.

## Signal Node Index Remapping [CONFIRMED]

send_signal의 경우:

```
IR:  args[0] = signal name (ClientExec)
     args[1] = first arg (InParam)

IR toIndex = 1 (args[1]이므로)
-> remapInputIndexForHiddenPin('send_signal', 1)
-> return idx - 1 = 0
-> GIA InParam[0]과 매칭
```

- IR에서 `args[0]`이 신호 이름이므로 첫 인자의 toIndex는 1
- GIA에서 InParam은 0부터 시작 (ClientExec와 별도 네임스페이스)
- 따라서 `idx - 1` 리매핑 필요

## Sources

- `gia.proto.ts` — NodeConnection 메시지 정의
- `gia_gen/basic.ts` — node_connect_from/to 함수
- `gia_gen/graph.ts` — Pin.encode() 연결 매칭 로직
- `ir_to_gia_transform/index.js` — remapInputIndexForHiddenPin
