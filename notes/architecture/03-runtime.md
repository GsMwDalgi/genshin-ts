# 03. 런타임 시스템

## 개요

`src/runtime/`은 Stage 2의 핵심이다. `.gs.ts` 파일이 서브프로세스에서 실행될 때 import되어 사용되는 모듈들로, 사용자가 작성한 `g.server().on(...)` 호출을 **실행 플로우(ExecutionFlow)**로 기록하고 최종적으로 IR JSON으로 직렬화한다.

## g.server() API

**파일:** `src/runtime/core.ts`

`g.server(opts)` 호출이 반환하는 `ServerGraphApi` 객체를 생성한다.

### 주요 옵션

| 옵션 | 타입 | 설명 |
|------|------|------|
| `id` | `number` | 대상 NodeGraph ID (기본 `1073741825`) |
| `name` | `string` | 그래프 표시 이름 (기본: 파일명) |
| `prefix` | `boolean` | `_GSTS_` 접두사 자동 추가 (기본 `true`) |
| `mode` | `'beyond' \| 'classic'` | 노드 그래프 모드 |
| `type` | `ServerGraphSubType` | `entity`, `status`, `class`, `item` |
| `variables` | `VariablesDefinition` | 그래프 변수 선언 |

### .on() 이벤트 핸들러

```typescript
g.server(opts).on('whenEntityIsCreated', (evt, f) => {
  // evt: 이벤트 출력 핀 접근
  // f: 노드 함수 호출 (380+ 메서드)
})
```

`.on()` 실행 시 내부적으로:
1. 이벤트 이름 정규화 (중국어 → 영어 별칭 변환)
2. 이벤트 메타데이터 조회 (`ServerEventMetadata`)
3. 전역 컨텍스트 설정 (`installScopedServerGlobals`)
4. 핸들러 실행 → 노드와 연결을 `ExecutionFlow`에 누적
5. `.on()` 체이닝 가능 (동일 인스턴스 반환)

### .onSignal() (포크 추가)

시그널 이벤트용 특수 메서드. `monitorSignal` 이벤트 타입을 사용하며 커스텀 시그널 인자 배열을 선택적으로 받는다.

## 값 타입 시스템

**파일:** `src/runtime/value.ts`

모든 노드 인자는 `value` 클래스의 인스턴스다.

### 타입 계층

```
value (base)
├── 기본: bool, int, float, str, vec3
├── 고급: guid, entity, prefabId, configId, faction
├── 컬렉션: list, dict, dictLiteral, ReadonlyDict
├── 특수: struct, generic, enumeration
└── 변수: localVariable, customVariableSnapshot
```

### 이중 역할

각 `value` 인스턴스는 두 가지로 사용된다:
- **리터럴 값**: `int(5)`, `str("hello")` → IR의 `Argument`로 직렬화
- **연결 참조**: 노드 함수 반환값 → IR의 `ConnectionArgument`로 직렬화 (어떤 노드의 몇 번째 출력 핀에서 왔는지 추적)

## 실행 플로우

**파일:** `src/runtime/execution_flow_types.ts`

핸들러 하나의 실행 플로우를 표현하는 핵심 자료 구조:

```
ExecutionFlow
  ├── eventNode        이벤트 노드 (시작점)
  ├── execNodes[]      실행 노드 목록 (순서 있음)
  ├── dataNodes[]      데이터 노드 목록
  ├── edges{}          실행 연결 (nodeId → 다음 노드들)
  └── execContextStack 분기 컨텍스트 스택
```

`ExecContext`는 분기(if/loop) 시 현재 실행 체인의 끝(tail)을 추적한다. 새 실행 노드가 추가되면 현재 컨텍스트의 `tailEndpoints`에서 새 노드로 연결이 생성된다.

## IR 빌더

**파일:** `src/runtime/ir_builder.ts`

`buildIRDocument()`가 여러 `ExecutionFlow`를 하나의 `IRDocument`로 조립한다.

### 빌드 과정

1. 각 ExecutionFlow의 이벤트/실행/데이터 노드 수집
2. 노드 ID 충돌 검증
3. `MetaCallRecord`의 인자를 `Argument` 타입으로 변환
4. 실행 연결을 `NextConnection` 형태로 변환
5. 변수 선언 수집
6. `IRDocument` 조립

### 시그널 노드 빌더 (포크 추가)

`buildSignalNode` 함수가 `send_signal`과 `monitor_signal` 노드에 `signalParams` 메타데이터를 포함시킨다.

## 전역 게임 API

**파일:** `src/runtime/server_globals.ts`

Stage 2 실행 시 전역 스코프에 설치되는 API:

- `player(n)` — 플레이어 entity
- `stage`, `level`, `self` — 특수 entity
- `Math`, `Vector3`, `Random` — 수학 API
- `int`, `float`, `str`, `bool`, `vec3` 등 — 값 생성자
- `list(type, [...])`, `dict(...)` — 컬렉션 생성자
- `send(signalName, args?)` — 시그널 전송 (포크에서 args 추가)
- `print(string)` — 디버그 출력

## 그래프 변수

**파일:** `src/runtime/variables.ts`

`g.server({ variables: { counter: int(0) } })` 선언으로 그래프 변수를 정의한다. `f.get('counter')` / `f.set('counter', value)` 메서드로 접근하며, 내부적으로 `get_node_graph_variable` / `set_node_graph_variable` 노드를 생성한다.
