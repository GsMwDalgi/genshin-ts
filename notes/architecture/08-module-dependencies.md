# 08. 모듈 의존성 및 빌드

## 개요

genshin-ts의 모듈 간 의존성을 레이어별로 정리한다. 서브프로세스 경계, 순환 의존성 주의점, 공개 API 경계, 그리고 빌드 출력 구조를 함께 기술한다.

## 레이어 구조

```
┌─────────────────────────────────────────────┐
│  CLI (src/cli/)                              │  사용자 진입점
├─────────────────────────────────────────────┤
│  Compiler Pipeline (src/compiler/)           │  파이프라인 오케스트레이션
├─────────────────────────────────────────────┤
│  AST Transform (ts_to_gs_transform/)         │  Stage 1
│  IR Builder (src/runtime/)                   │  Stage 2
│  GIA Transform (ir_to_gia_transform/)        │  Stage 3
├─────────────────────────────────────────────┤
│  Definitions (src/definitions/)              │  게임 API 정의
├─────────────────────────────────────────────┤
│  Injector (src/injector/)                    │  .gil 바이너리 처리
├─────────────────────────────────────────────┤
│  Shared / I18n (src/shared/, src/i18n/)      │  공유 유틸리티
├─────────────────────────────────────────────┤
│  Thirdparty (src/thirdparty/)                │  외부 패키지
└─────────────────────────────────────────────┘
```

## 서브프로세스 경계

두 곳에서 서브프로세스가 spawn된다. 이 경계는 모듈 의존성의 단절점이다:

| spawn 위치 | 서브프로세스 | 접근 모듈 |
|------------|------------|----------|
| `gs_to_ir_json_transform/index.ts` | `runner.ts` | `src/runtime/` 전체 |
| `ir_to_gia_pipeline.ts` | `ir_to_gia_transform/runner.ts` | `ir_to_gia_transform/` |

서브프로세스는 부모 프로세스와 메모리를 공유하지 않는다. 데이터 전달은 파일 시스템(JSON)과 환경 변수를 통해서만 이루어진다.

## 순환 의존성 주의점

### definitions/nodes.ts ↔ runtime/core.ts

상호 의존이지만 TypeScript 타입만 import하므로 런타임 순환은 발생하지 않는다. 값 import로 바꾸면 문제가 생길 수 있다.

### ts_to_gs_transform/stmt.ts ↔ expr.ts

실제 순환 의존이다. `stmt.ts`가 `expr.ts`의 `transformExpression`을 import하고, `expr.ts`가 `stmt.ts`의 `transformHandler`를 import한다 (중첩 핸들러 처리용). 현재 ESM 초기화 순서상 문제없으나, 초기화 코드에 부작용 추가 시 주의 필요.

## 공개 API 경계

`src/index.ts`가 외부 공개 API를 정의한다:

- **컴파일러:** `compileTsToGs`, `emitIrJsonForEntries`, `writeGiaFromIrJsonFiles` 등
- **인젝터:** `createInjector`, `injectGilBytes`, `injectGilFile` 및 관련 타입
- **정의:** `prefabs.ts` 재export

사용자 프로젝트에서 import하는 타입(`GstsConfig`, 런타임 API 등)은 `types/gsts/index.d.ts`에서 제공된다 (`postbuild.mjs`가 빌드 후 자동 생성).

## Thirdparty 패키지

`src/thirdparty/Genshin-Impact-Miliastra-Wonderland-Code-Node-Editor-Pack/`

MIT 라이선스의 외부 모듈. genshin-ts가 직접 통합한 것.

| 파일/디렉터리 | 역할 |
|--------------|------|
| `gia_gen/graph.ts` | `Graph`, `Node`, `Pin` 빌더 클래스 |
| `node_data/node_id.ts` | `NODE_ID` — 노드 타입 → 게임 내부 ID |
| `node_data/enum_id.ts` | `ENUM_ID`, `ENUM_VALUE` — 열거형 ID |
| `protobuf/decode.ts` | `wrap_gia()` — GIA 프로토버프 직렬화 |
| `protobuf/gia.proto.ts` | GIA 프로토버프 스키마 |

`src/compiler/gia_vendor.ts`가 이들을 re-export한다 (`// @ts-nocheck` 적용).

## 빌드 출력

```
dist/
  src/           컴파일된 JavaScript + 타입 정의
types/
  gsts/
    index.d.ts   사용자 프로젝트용 타입 (postbuild.mjs 자동 생성)
bin/
  gsts.mjs       CLI 진입점
```

### 빌드 명령

| 명령 | 설명 |
|------|------|
| `npm run build` | TypeScript 컴파일 + postbuild |
| `npm run gen` | 정의 파일 코드 생성 |
| `npm test` | 빌드 + 통합 테스트 |
| `npm run pack` | 패키징 |

## Windows 경로 처리

genshin-ts는 Windows를 주요 타겟으로 한다. 경로 처리 시:
- `path.posix.join()` 또는 `toPosixPath()` 사용
- `\` 대신 `/` 사용 (fast-glob 호환성)
- `path.resolve()`로 정규화 후 사용
