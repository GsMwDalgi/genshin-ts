# Changelog — 포크 변경사항

> **포크 원본:** josStorer/genshin-ts v0.1.7
> **문서 작성일:** 2026-03-31

이 문서는 upstream(josStorer/genshin-ts v0.1.7)에서 포크한 뒤 적용된 모든 변경사항을 정리한다. 향후 upstream 업데이트를 머지할 때 충돌 지점을 빠르게 파악하기 위한 참조용이다.

---

## 목차

1. [시그널 커스텀 인자 (Signal Arguments)](#1-시그널-커스텀-인자)
2. [인젝터 varint 오버플로우 수정](#2-인젝터-varint-오버플로우-수정)
3. [isEntityLikeType `any` 타입 가드 추가](#3-isentityliketype-any-타입-가드-추가)
4. [GIL Inspector CLI 명령 (`gsts inspect`)](#4-gil-inspector-cli-명령)
5. [GIL Scaffold CLI 명령 (`gsts scaffold`)](#5-gil-scaffold-cli-명령)
6. [GIL Reader 모듈](#6-gil-reader-모듈)
7. [.gitignore 업데이트](#7-gitignore-업데이트)

---

## 1. 시그널 커스텀 인자

**카테고리:** 기능 추가
**영향 범위:** 런타임, 컴파일러, 인젝터, 정의

`sendSignal`과 `onSignal`에 최대 18종의 커스텀 인자를 전달할 수 있도록 기능을 추가했다. 기존에는 시그널 이름만 전달 가능했으나, 이제 `entity`, `int`, `str`, `vec3` 등의 값을 시그널에 실어 보낼 수 있다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/runtime/value.ts` | `SignalArgDef`, `SignalArgsToPayload` 타입 추가 |
| `src/runtime/core.ts` | `onSignal` 핸들러에서 시그널 인자 타입을 동적으로 처리하도록 확장 |
| `src/runtime/ir_builder.ts` | `buildSignalNode` 빌더 추가 — `send_signal`, `monitor_signal` 노드에 `signalParams` 포함 |
| `src/runtime/meta_call_types.ts` | `MetaCallRecord`에 `signalParams` 필드 추가 |
| `src/runtime/IR.d.ts` | `Node` 인터페이스에 `signalParams` 필드 추가 |
| `src/runtime/server_globals.ts` | `send()` 전역 함수에 `args` 매개변수 추가 |
| `src/runtime/server_globals.d.ts` | `send()` 타입 시그니처에 `args?` 추가 |
| `src/definitions/nodes.ts` | `sendSignal` 메서드에 `signalArgs` 배열 매개변수 추가, 배열 자동 래핑 로직 포함 |
| `src/definitions/events-payload.ts` | `value` 임포트 추가 (시그널 페이로드 타입용) |
| `src/compiler/ir_to_gia_transform/mappings.ts` | `SIGNAL_ARG_TYPE_MAP` 상수 추가 — 18종 시그널 인자 타입의 `typeId`/`dataGroup` 매핑 |
| `src/compiler/ir_to_gia_transform/index.ts` | `send_signal` 노드의 시그널 인자 핀 생성 로직 추가, `send_signal` 인덱스 매핑 추가 |
| `src/compiler/ir_to_gia_transform/pins.ts` | `ensureInputPinWithType()` 함수 추가, `send_signal`의 entity null 처리 추가 |
| `src/injector/signal_nodes.ts` | `extractSignalNameFromNode`에서 ClientExec 핀(kind=5) 우선 탐색 로직 추가 |

### 사용 예시

```typescript
// 시그널 보내기
send(str('my_signal'), {
  targetEntity: entity(1),
  damageAmount: int(100)
})

// 시그널 받기 — onSignal에서 evt로 인자에 접근
g.server().onSignal('my_signal', [
  { name: 'targetEntity', type: 'entity' },
  { name: 'damageAmount', type: 'int' }
], (evt, f) => {
  // evt.targetEntity, evt.damageAmount 사용 가능
})
```

---

## 2. 인젝터 varint 오버플로우 수정

**카테고리:** 버그 수정
**영향 범위:** 인젝터

`src/injector/signal_nodes.ts`의 프로토버프 varint 파싱 함수들에서 32비트 정수 오버플로우로 인해 인젝션이 무한 루프에 빠지는 문제를 수정했다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/injector/signal_nodes.ts` | `readFieldMessages`, `readFieldBytes`, `parseNodeGraphId` 함수에 오프셋 음수/오버플로우 검사 추가 |

### 수정 내용

- `while (offset < buf.length)` → `while (offset >= 0 && offset < buf.length)` : 오프셋이 음수가 되면 루프 탈출
- `if (len < 0) break` : 파싱된 길이가 음수인 경우 조기 탈출
- `if (dataEnd > buf.length || dataEnd < 0) break` : 데이터 끝 위치 오버플로우 검사
- `if (newOffset < 0) break` : 새 오프셋 계산 결과 오버플로우 검사

---

## 3. isEntityLikeType `any` 타입 가드 추가

**카테고리:** 버그 수정
**영향 범위:** 컴파일러 (공유 유틸리티)

`onSignal` 핸들러에서 시그널 인자를 중간 변수에 할당하면 `entity` 타입으로 오추론되는 문제를 수정했다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/shared/ts_type_utils.ts` | `isEntityLikeType()` 함수 시작부에 `any` 타입 조기 반환 추가 |

### 원인 및 수정

`evt`가 `...& Record<string, any>` 타입이므로 `evt.score` 같은 접근은 `any` 타입을 반환한다. TypeScript의 `checker.isTypeAssignableTo(any, entity)`는 항상 `true`를 반환하므로, `isEntityLikeType`이 `any`를 `entity`로 잘못 판단했다.

```diff
 ): boolean {
+  if (type.flags & ts.TypeFlags.Any) return false
   const aliasName = type.aliasSymbol?.getName() ?? type.symbol?.getName()
```

이 수정이 다른 문제를 유발할 경우 `src/shared/ts_type_utils.ts:28`의 `if (type.flags & ts.TypeFlags.Any) return false` 줄을 삭제하면 롤백된다. 롤백 후에는 시그널 인자를 중간 변수 없이 `evt.argName` 형태로 직접 인라인 사용해야 한다.

---

## 4. GIL Inspector CLI 명령

**카테고리:** 기능 추가 (개발 도구)
**영향 범위:** CLI

`.gil` 맵 파일의 NodeGraph 정보를 CLI에서 조회할 수 있는 `gsts inspect` 명령을 추가했다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/cli/gil_inspect.ts` | 새 파일 — inspect 명령 구현 (153줄) |
| `src/cli/gsts.ts` | `inspect` 서브 명령 등록 |
| `src/i18n/locales/en-US/main.json` | inspect 관련 i18n 문자열 추가 |
| `src/i18n/locales/zh-CN/main.json` | inspect 관련 i18n 문자열 추가 |

### 기능

- `gsts inspect <file>` — 모든 NodeGraph의 요약 테이블 출력 (ID, 이름, 타입, 노드 수, 변수 수)
- `gsts inspect <file> --id <id>` — 특정 NodeGraph의 상세 정보 출력 (변수, 노드, 연결 체인)
- `--json` 옵션으로 JSON 출력, `--raw` 옵션으로 원시 프로토버프 디코딩 결과 출력

---

## 5. GIL Scaffold CLI 명령

**카테고리:** 기능 추가 (개발 도구)
**영향 범위:** CLI

`.gil` 파일의 기존 NodeGraph를 읽어 genshin-ts TypeScript 스캐폴드(scaffold)를 생성하는 `gsts scaffold` 명령을 추가했다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/cli/gil_scaffold.ts` | 새 파일 — scaffold 명령 구현 (236줄) |
| `src/cli/gsts.ts` | `scaffold` 서브 명령 등록 |
| `src/i18n/locales/en-US/main.json` | scaffold 관련 i18n 문자열 추가 |
| `src/i18n/locales/zh-CN/main.json` | scaffold 관련 i18n 문자열 추가 |

### 기능

- `gsts scaffold <file> --id <id>` — 지정된 NodeGraph의 변수 선언과 이벤트 핸들러 스캐폴드를 `.ts` 파일로 생성
- `--out <file>` — 출력 경로 지정 (기본: 표준 출력)
- `--force` — 기존 파일 덮어쓰기

---

## 6. GIL Reader 모듈

**카테고리:** 기능 추가 (내부 모듈)
**영향 범위:** 인젝터

`.gil` 바이너리에서 NodeGraph의 상세 정보를 읽어오는 범용 리더 모듈을 추가했다. `inspect`과 `scaffold` 명령의 공통 기반이다.

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `src/injector/reader.ts` | 새 파일 — GIL NodeGraph 리더 (367줄) |

### 주요 기능

- `readGilNodeGraphs(gilBytes)` — 모든 NodeGraph의 요약 정보 목록 반환 (`NodeGraphSummary[]`)
- `readGilNodeGraph(gilBytes, targetId)` — 특정 NodeGraph의 상세 정보 반환 (`NodeGraphDetail`)
- 노드 타입 역참조 (게임 내부 ID → camelCase 이름), 변수 정보 파싱, 실행 연결 체인 파싱

---

## 7. .gitignore 업데이트

**카테고리:** 설정
**영향 범위:** 프로젝트 설정

### 변경 파일

| 파일 | 변경 내용 |
|------|----------|
| `.gitignore` | `.vs/` (Visual Studio) 및 `.claude/` (Claude Code) 디렉터리 제외 추가 |
