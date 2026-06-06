# 02. 컴파일 파이프라인

## 개요

genshin-ts의 컴파일 파이프라인은 TypeScript 소스를 3단계에 걸쳐 게임이 인식하는 GIA 바이너리로 변환한다. 각 단계는 독립된 Node.js 서브프로세스로 실행되며, 사이드 이펙트 격리와 병렬 처리를 위해 프로세스가 분리되어 있다.

## 전체 흐름

```
Stage 1: .ts  →  .gs.ts      TypeScript AST 변환
Stage 2: .gs.ts  →  .json    런타임 실행 → IR 직렬화
  (선택) IR 병합              동일 graph.id를 가진 IR 파일 병합
Stage 3: .json  →  .gia      IR → GIA 프로토버프 변환
```

## Stage 1: TypeScript → GenshinScript

**진입점:** `src/compiler/ts_to_gs_pipeline.ts` → `compileTsToGs()`

사용자의 `.ts` 파일을 TypeScript Compiler API로 AST 변환하여 `.gs.ts` 파일을 생성한다. `.gs.ts`는 genshin-ts 런타임 API를 직접 호출하는 유효한 TypeScript 파일이다.

### 처리 흐름

1. `gsts.config.ts`의 `entries` 패턴을 `fast-glob`으로 확장하여 입력 파일 목록 생성
2. `ts.createProgram()`으로 TypeScript 프로그램 생성 (타입 체커 획득)
3. 전체 소스파일의 타이머 호출 수를 사전 스캔하여 파일별 타이머 인덱스 오프셋 계산
4. 각 파일에 대해 `transformToGs()` 호출 → AST 변환
5. `.gs.ts` 파일 출력 (원본 디렉터리 구조 유지)

### 변환 범위

`g.server().on(...)` 콜백 내부와 `gstsServer*` 이름의 함수 내부만 변환된다. 최상위 스코프의 코드는 변환 없이 그대로 유지된다.

### AST 변환 규칙 요약

| TypeScript | 변환 결과 |
|------------|----------|
| `const x = expr` | 단순 대입 또는 `localVariable` 노드 |
| `let x = expr` | `localVariable` 초기화 + `setLocalVariable` |
| `if (cond) {...}` | `f.branch(cond, trueHandler, falseHandler)` |
| `for (const x of list) {...}` | `f.forList(list, (x) => {...})` |
| `for (let i=0; i<n; i++) {...}` | `f.finiteLoop(start, end, (i) => {...})` |
| `while (cond) {...}` | `f.whileCondition(cond, body)` (feature flag 필요) |
| `Math.abs(x)` | `f.abs(x)` |
| `arr.push(x)` | `f.appendToList(arr, x)` |
| `console.log(x)` | `f.printString(f.toString(x))` |
| `setTimeout(cb, ms)` | 타이머 풀 시스템으로 변환 |

### 주요 파일

| 파일 | 역할 |
|------|------|
| `ts_to_gs_transform/index.ts` | 진입점, 최상위 방문자 |
| `ts_to_gs_transform/stmt.ts` | 구문 변환 (if, for, return 등) |
| `ts_to_gs_transform/expr.ts` | 표현식 변환 (연산자, 함수 호출 등) |
| `ts_to_gs_transform/loops.ts` | 루프 변환 (for, while, do-while) |
| `ts_to_gs_transform/builtins.ts` | 빌트인 API 변환 (Math, Vector3 등) |
| `ts_to_gs_transform/list_methods.ts` | 배열 메서드 변환 (push, length 등) |
| `ts_to_gs_transform/types.ts` | 변환 컨텍스트 타입 및 feature flags |

## Stage 2: GenshinScript → IR JSON

**진입점:** `src/compiler/gs_to_ir_json_transform/index.ts` → `emitIrJsonForEntries()`

`.gs.ts` 파일을 서브프로세스에서 **실제로 실행**한다. 런타임 모듈이 "무엇을 해야 하는가"를 기록(record)하고, 실행 완료 후 IR JSON으로 직렬화한다.

### 처리 흐름

1. 각 엔트리 `.gs.ts` 파일에 대해 서브프로세스(`runner.js`) spawn
2. runner가 해당 파일을 `import`하여 실행
3. `g.server().on(...)` 호출들이 내부 실행 플로우를 누적
4. `buildIRDocument()`가 `ExecutionFlow`를 IR JSON으로 직렬화
5. `.json` 파일로 저장

### 서브프로세스 격리 이유

- ESM 모듈 캐시 오염 방지
- 사이드 이펙트 격리
- 병렬 처리 (`os.cpus().length - 1` 동시 실행)

### IR 병합

동일한 `graph.id`를 가진 여러 IR JSON 파일이 있으면 병합한다. 노드 ID를 재번호화하여 충돌을 방지한다.

## Stage 3: IR JSON → GIA

**진입점:** `src/compiler/ir_to_gia_pipeline.ts` → `writeGiaFromIrJsonFiles()`

IR JSON을 게임의 `.gia` 프로토버프 바이너리로 변환한다.

### 처리 흐름

1. IR 노드 정규화 및 전처리 (`expandListLiterals`)
2. 타이머 분산 집계 최적화 (선택적)
3. 연결 타입 인덱스 빌드
4. 에디터 내 노드 위치 자동 계산 (BFS 기반 토폴로지 정렬)
5. `Graph` / `Node` / `Pin` 빌더로 GIA 프로토버프 조립
6. `wrap_gia()` → 20바이트 헤더 + 프로토버프 바이트 + 4바이트 푸터

### 최적화

| 최적화 | 설명 |
|--------|------|
| 타이머 분산 집계 | 여러 타이머의 if-else 체인을 switch 노드로 집약 (최대 10 case) |
| 데드 노드 제거 | 이벤트에 연결되지 않은 실행/데이터 노드 제거 |
| 상수 접기 (precompile) | 컴파일 타임에 결정 가능한 표현식 사전 계산 |

## dev 모드 증분 빌드

`gsts dev` 명령으로 `chokidar` 기반 파일 감시를 실행한다:

- 변경된 파일만 Stage 1 재실행
- 엔트리 파일만 Stage 2/3 재처리
- `.gil` 파일이 외부(에디터)에서 저장되면 자동 재주입 (`reinjectOnMapChange: true`)
