# GIL Reverse Analysis (GIL -> TS scaffold)

`gsts inspect` / `gsts scaffold` 구현 과정에서 확인된 사실.

## Event Entry Node Identification [CONFIRMED]

GIL에서 디코딩한 NodeGraph에서 이벤트 핸들러의 시작점(이벤트 엔트리 노드)을 식별하는 방법:

### Rule

**OutFlow 핀을 가지고 있으면서, 다른 어떤 노드의 OutFlow 연결 대상이 아닌 노드.**

단순히 "InFlow 핀이 없는 노드"로 판별하면 데이터 전용 노드(getNodeGraphVariable 등)도
이벤트 엔트리로 잘못 감지된다. 반드시 2-pass 접근이 필요하다:

1. 모든 노드의 OutFlow 연결 대상 nodeIndex를 수집 (InFlow 타겟 집합)
2. OutFlow 핀이 있으면서 InFlow 타겟 집합에 포함되지 않는 노드 = 이벤트 엔트리

```ts
// Pass 1: InFlow 타겟 수집
const inflowTargets = new Set<number>()
for (const node of allNodes) {
  for (const pin of node.pins) {
    if (pin.i1?.kind === 'OutFlow') {
      for (const conn of pin.connects ?? []) {
        inflowTargets.add(conn.id)
      }
    }
  }
}

// Pass 2: 이벤트 엔트리 판정
for (const node of allNodes) {
  const hasOutFlow = node.pins.some(p => p.i1?.kind === 'OutFlow')
  const isEventEntry = hasOutFlow && !inflowTargets.has(node.nodeIndex)
}
```

### Reasoning

- 데이터 노드(getNodeGraphVariable, addition 등): OutFlow 핀 없음 -> 이벤트 엔트리 아님
- 중간 실행 노드(printString, setNodeGraphVariable 등): OutFlow 핀 있지만 다른 노드의 대상 -> 아님
- whenEntityIsCreated, whenTimerIsTriggered: OutFlow 있고 아무도 연결 안 함 -> 이벤트 엔트리

## Signal Node Reverse Identification [CONFIRMED]

### Structure

시그널 노드는 인젝터가 패치한 높은 범위의 nodeId를 가짐 (1610612737 등).
NODE_ID 테이블에 없으므로 일반 역조회로는 이름을 알 수 없다.

### Identification Key

```
genericId.kind === 'SysGraph'   // 시그널 또는 특수 그래프 노드
```

일반 노드는 `kind: 'SysCall'`이고, 시그널 노드만 `kind: 'SysGraph'`이다.

### Signal Name Extraction

`ClientExecNode` kind 핀의 문자열 값이 시그널 이름:

```ts
for (const pin of node.pins) {
  if (pin.i1?.kind === 'ClientExecNode') {
    const signalName = pin.value?.bString?.val  // 'NextEffect'
  }
}
```

### send vs monitor Distinction

| Role                         | Has OutFlow | Is InFlow Target |
|------------------------------|-------------|------------------|
| monitor_signal (onSignal)    | Yes         | No (event entry) |
| send_signal (sendSignal)     | No          | Yes              |

send_signal은 실행 흐름 안에서 호출되므로 InFlow를 받고,
monitor_signal은 이벤트 핸들러 시작점이므로 InFlow를 받지 않는다.

## Variable Type Mapping [CONFIRMED]

GIL의 `GraphVariable.type` (VarType enum) -> TS scaffold 타입:

| VarType | Name           | TS Constructor       | Default Example  |
|---------|----------------|----------------------|------------------|
| 3       | int            | `int()`              | `int(0)`         |
| 4       | bool           | `bool()`             | `bool(false)`    |
| 5       | float          | `float()`            | `float(0)`       |
| 6       | str            | `str()`              | `str("")`        |
| 20      | config_id      | `configId()`         | `configId(2)`    |
| 8       | int_list       | `/* int_list */`     | 초기값 추출 불가  |
| 22      | config_id_list | `/* config_id_list */` | 초기값 추출 불가 |
| 27      | dict           | `/* dict */`         | 초기값 추출 불가  |

리스트/딕셔너리 타입의 초기값은 GIL 바이너리에 inline 인코딩되어 있어
현재 reader.ts에서는 타입만 표시하고 초기값은 주석으로 남긴다.

## Compiler-Generated Internal Variables [CONFIRMED]

원본 TS에 없지만 GIL에 존재하는 변수들:

| Pattern                   | Cause                         | Example                       |
|---------------------------|-------------------------------|-------------------------------|
| `__gsts_timeout_N_*`     | `setTimeout()` 구현용 dict/index | `__gsts_timeout_0_cap_mySeq` |
| `__gsts_stage`           | `stage` 전역 변수 캐싱          | `__gsts_stage: entity(0)`    |

scaffold에서 이 변수들이 보이면 원본 TS에서 setTimeout이나 stage를 사용한 것이다.

## Sources

- `genshin-ts/src/injector/reader.ts` — GIL 파싱 API
- `genshin-ts/src/cli/gil_inspect.ts` — inspect 명령어
- `genshin-ts/src/cli/gil_scaffold.ts` — scaffold 명령어
- SampleForEffect / test-compile-func 인젝션-추출 비교 (2026-03-29)
