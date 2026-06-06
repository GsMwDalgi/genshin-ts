# 04. IR JSON 포맷

## 개요

IR(Intermediate Representation) JSON은 Stage 2와 Stage 3 사이의 중간 표현이다. 노드 그래프의 모든 정보를 자기 기술적(self-describing) JSON 형식으로 담고 있으며, 디버깅과 추가 처리를 위한 핵심 결과물이다.

**타입 정의 위치:** `src/runtime/IR.d.ts`

## 최상위 구조

```typescript
type IRDocument = {
  ir_version: 1
  ir_type: 'node_graph'
  graph: ServerGraphInfo     // 그래프 메타 정보
  variables?: Variable[]     // 그래프 변수 선언
  nodes?: ServerNode[]       // 노드 목록
}
```

## 그래프 메타 (ServerGraphInfo)

```typescript
{
  name?: string              // 에디터 표시 이름 (_GSTS_ 접두사 자동)
  id?: number                // NodeGraph ID (기본 1073741825)
  type: 'server'
  mode?: 'beyond' | 'classic'
  sub_type?: 'entity' | 'status' | 'class' | 'item'
}
```

## 노드 (ServerNode)

```typescript
{
  id: number                 // 파일 내 고유 ID
  type: string               // 노드 타입 (camelCase, 예: "whenEntityIsCreated")
  args?: Argument[]          // 입력 인자
  next?: NextConnection[]    // 실행 출력 연결
  position?: [number, number] // 에디터 내 위치 (자동 계산)
  signalParams?: Array<{ name: string; type: string }>  // 시그널 인자 (포크 추가)
}
```

## 인자 타입 (Argument)

노드의 각 입력 핀에 해당한다. 두 가지 종류가 있다:

### 리터럴 값

```json
{ "type": "int", "value": 5 }
{ "type": "str", "value": "hello" }
{ "type": "bool", "value": true }
{ "type": "vec3", "value": [1.0, 2.0, 3.0] }
{ "type": "enum", "value": "Ascending" }
```

지원 타입: `bool`, `int`, `float`, `str`, `vec3`, `guid`, `prefab_id`, `config_id`, `faction`, `enum`, `enumeration`, `dict`

### 연결 참조 (ConnectionArgument)

다른 노드의 출력 핀에서 값을 가져온다:

```json
{
  "type": "conn",
  "value": {
    "node_id": 3,          // 소스 노드 ID
    "index": 0,            // 소스 노드의 출력 핀 인덱스 (0-based)
    "type": "int"          // 데이터 타입
  }
}
```

## 실행 연결 (NextConnection)

노드의 실행 출력 핀에서 다음 노드로의 연결.

- **단순 형태:** 숫자 하나 → `source_index=0`에서 해당 노드로 연결
- **상세 형태:** 다중 실행 출력이 있는 노드(branch 등)에서 사용

```json
"next": [
  { "node_id": 10, "source_index": 0 },
  { "node_id": 20, "source_index": 1 }
]
```

`branch` 노드에서 `source_index: 0`은 true 분기, `source_index: 1`은 false 분기.

## 변수 (Variable)

```json
{ "name": "counter", "type": "int", "value": 0 }
{ "name": "items", "type": "str_list", "length": 5 }
{ "name": "lookup", "type": "dict", "dict": { "k": "str", "v": "int" } }
```

## 특수 IR 형식

### 배열 IR (멀티 그래프)

하나의 `.json` 파일이 `IRDocument[]` 배열을 담을 수 있다. 동일 파일에서 여러 `g.server()`를 다른 ID로 선언한 경우 발생한다. Stage 3에서 각 문서가 별도 `.gia` 파일로 출력된다.

### 병합된 IR

`ir_merge.ts`가 생성하는 병합 IR은 `__gsts` 메타 필드를 포함한다:

```json
{
  "__gsts": {
    "merged": true,
    "graphId": 1073741825,
    "sources": ["dist/src/a.json", "dist/src/b.json"]
  }
}
```

이 메타로 중복 병합을 방지한다.

## GIA 파일 구조

`.gia`는 프로토버프 인코딩된 NodeGraph 바이너리로, 다음과 같이 감싸진다:

```
[20 bytes header] [protobuf NodeGraph bytes] [4 bytes footer]
```
