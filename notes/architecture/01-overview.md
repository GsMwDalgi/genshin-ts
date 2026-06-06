# 01. 프로젝트 개요

## genshin-ts란

genshin-ts는 원신(Genshin Impact)의 Miliastra Wonderland 에디터용 노드 그래프를 TypeScript로 작성할 수 있게 해주는 컴파일러 프로젝트다. 비주얼 노드 그래프 에디터 대신 TypeScript 코드로 게임 로직을 작성하면, 컴파일러가 이를 게임이 인식하는 `.gia` 바이너리 형식으로 변환하고 맵 파일(`.gil`)에 주입한다.

## 핵심 파이프라인

```
TypeScript (.ts)
   ↓  Stage 1: AST 변환 (ts_to_gs_transform)
GenshinScript (.gs.ts)
   ↓  Stage 2: 런타임 실행 → IR 직렬화 (gs_to_ir_json_transform)
IR JSON (.json)
   ↓  (선택) IR 병합 (ir_merge)
   ↓  Stage 3: IR → 프로토버프 (ir_to_gia_transform)
GIA Binary (.gia)
   ↓  인젝터 (injector)
GIL Map File (.gil)
```

각 단계는 독립된 Node.js 서브프로세스로 실행된다. 이는 Stage 2에서 `.gs.ts` 파일을 실제로 `import`하여 실행하기 때문으로, 사이드 이펙트 격리와 ESM 모듈 캐시 오염 방지를 위한 설계다.

## 디렉터리 구조

```
src/
  cli/              CLI 명령 구현 (gsts, dev, maps, inspect, scaffold)
  compiler/         컴파일 파이프라인
    ts_to_gs_transform/   Stage 1: TS AST → .gs.ts 변환
    gs_to_ir_json_transform/  Stage 2: .gs.ts 실행 → IR JSON 생성
    ir_to_gia_transform/      Stage 3: IR JSON → GIA 바이너리 변환
  runtime/          Stage 2 런타임 (g.server() API, 값 타입, IR 빌더)
  definitions/      게임 노드 API 정의 (이벤트, 함수, 열거형)
  injector/         GIA → GIL 바이너리 패치
  eslint/           genshin-ts 전용 ESLint 규칙
  i18n/             CLI 국제화 (en-US, zh-CN)
  shared/           공유 유틸리티
  thirdparty/       외부 패키지 (GIA 프로토버프, 노드 ID 룩업)

resources/          노드 정의 원시 데이터 (node_definitions.json 등)
scripts/            코드 생성 및 빌드 스크립트
configs/            TypeScript 플러그인, tsconfig 프리셋
types/gsts/         사용자 프로젝트용 타입 정의 (빌드 후 자동 생성)
bin/                CLI 진입점 (gsts.mjs)
```

## 시스템 규모

| 서브시스템 | 주요 파일 수 | 대략적 규모 |
|-----------|------------|-----------|
| 노드 정의 (definitions) | `nodes.ts`, `enum.ts`, `events*.ts` 등 | ~26,000줄 |
| 런타임 (runtime) | `core.ts`, `value.ts`, `server_globals.ts` 등 | ~120,000줄 |
| 컴파일러 (compiler) | `ts_to_gs_transform/`, `ir_to_gia_transform/` | ~6,500줄 |
| 인젝터 (injector) | `binary.ts`, `node_graph.ts`, `reader.ts` 등 | ~2,000줄 |
| CLI | `gsts.ts`, `gil_inspect.ts`, `gil_scaffold.ts` 등 | ~2,000줄 |

## 지원 범위

- **서버 노드 그래프**: 완전 지원 (380+ 노드 메서드, 59개 이벤트, 51개 열거형)
- **그래프 모드**: beyond / classic
- **그래프 타입**: entity / status / class / item
- **시그널 시스템**: 커스텀 인자 포함 (포크 추가 기능)
- **클라이언트 노드 그래프**: 미지원 (IR 타입 정의만 존재)

## 기술 스택

- **언어:** TypeScript (ESM)
- **런타임:** Node.js v20+
- **빌드:** tsc + postbuild 스크립트
- **CLI 프레임워크:** commander
- **프로토버프:** protobufjs (GIA/GIL 직렬화)
- **타입 체커:** TypeScript Compiler API (AST 변환에 직접 사용)
- **감시 모드:** chokidar (파일 변경 감지)
