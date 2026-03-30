# 06. CLI

## 개요

`src/cli/`는 `gsts` CLI 명령의 구현부다. `bin/gsts.mjs`가 진입점이며, `src/cli/gsts.ts`가 모든 서브 명령을 정의한다. `commander` 라이브러리 기반.

## 전역 옵션

```
gsts [--config <path>] [--noinject] [--lang <lang>] <command>
```

| 옵션 | 설명 |
|------|------|
| `--config` | `gsts.config.ts` 경로 (기본: CWD의 `gsts.config.ts`) |
| `--noinject` | 주입 단계 건너뛰기 |
| `--lang` | CLI 출력 언어 강제 지정 |

## 서브 명령

| 명령 | 역할 |
|------|------|
| `gsts` (기본) | 단일 파일 컴파일 및 주입 |
| `gsts dev` | 증분 빌드 + 파일 감시 모드 |
| `gsts maps` | 최근 수정된 맵 파일 목록 출력 |
| `gsts inspect` | GIL 파일의 NodeGraph 정보 조회 (포크 추가) |
| `gsts scaffold` | GIL에서 TypeScript 코드 스캐폴드 생성 (포크 추가) |

## 배치 빌드 흐름

```
compileTsToGs()                    Stage 1
  → emitIrJsonForEntries()         Stage 2 (병렬)
    → mergeIrJsonFilesByGraphId()  병합 (필요 시)
      → writeGiaFromIrJsonFiles()  Stage 3 (병렬)
        → injectMany()             주입
```

## dev 모드

`chokidar`로 소스 파일 변경을 감시한다:

- 변경된 파일만 Stage 1 재실행
- 엔트리 파일만 Stage 2/3 재처리
- `.gil` 파일이 외부(에디터)에서 저장되면 자동 재주입

설정 파일(`gsts.config.ts`)은 `mtime` 기반으로 캐싱하여 변경 시에만 재로드한다.

## inspect 명령 (포크 추가)

**파일:** `src/cli/gil_inspect.ts`

`.gil` 파일의 NodeGraph 정보를 조회한다.

```bash
# 전체 NodeGraph 요약 테이블
gsts inspect path/to/map.gil

# 특정 NodeGraph 상세 (변수, 노드, 연결 체인)
gsts inspect path/to/map.gil --id 1073741825

# JSON 출력
gsts inspect path/to/map.gil --json

# 원시 프로토버프 디코딩 결과
gsts inspect path/to/map.gil --id 1073741825 --json --raw
```

## scaffold 명령 (포크 추가)

**파일:** `src/cli/gil_scaffold.ts`

기존 NodeGraph를 읽어 genshin-ts TypeScript 스캐폴드(scaffold)를 생성한다.

```bash
# 표준 출력으로 스캐폴드 출력
gsts scaffold path/to/map.gil --id 1073741825

# 파일로 저장
gsts scaffold path/to/map.gil --id 1073741825 --out src/main.ts
```

생성되는 스캐폴드에는 변수 선언과 이벤트 핸들러 골격이 포함된다.

## 설정 로더

**파일:** `src/compiler/config_loader.ts`

`gsts.config.ts`를 `tsx`로 동적 import하여 `GstsConfig` 객체를 반환한다. TypeScript 파일을 직접 실행할 수 있어 별도 컴파일이 불필요하다.

## GIL 경로 해석

**파일:** `src/cli/gil_paths.ts`

Windows에서 게임 설치 경로의 BeyondLocal 폴더를 찾는다:

```
%LocalAppData%/../LocalLow/miHoYo/
  ├── 原神/BeyondLocal/           (중국 서버)
  └── Genshin Impact/BeyondLocal/  (글로벌 서버)
```

`playerId`가 미지정이면 폴더 내 유일한 숫자 디렉터리를 자동 탐지한다.

## 커스텀 리소스 추출

**파일:** `src/cli/gil_resources.ts`

`.gil` 파일에서 Custom Prefab ID를 추출하여 `src/resources/prefabs.ts`로 저장한다. `extractResources: true` (기본값)일 때 빌드 시 자동 실행된다.

## 국제화

**파일:** `src/i18n/`

`i18next` 기반. CLI 메시지 번역 파일은 `src/i18n/locales/`에 있다 (`zh-CN`, `en-US`).

언어 결정 순서:
1. `--lang` CLI 플래그
2. `GstsConfig.lang` 설정값
3. 시스템 언어 자동 감지 (`os-locale` + 환경 변수)
4. fallback: `en-US`
