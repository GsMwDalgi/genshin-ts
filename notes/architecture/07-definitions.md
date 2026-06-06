# 07. 정의 파일 및 코드 생성

## 개요

`src/definitions/`의 TypeScript 파일들은 게임 노드 그래프 시스템의 API 정의를 담는다. 대부분은 `scripts/generate-definitions.ts`가 `resources/node_definitions.json`을 읽어 자동 생성한다.

**실행 명령:** `npm run gen`

## 자동 생성 파일

| 파일 | 내용 |
|------|------|
| `events.ts` | `ServerEventMetadata` — 59개 이벤트의 출력 핀 메타데이터 |
| `events-payload.ts` | `ServerEventPayloads` — 이벤트 핸들러의 `evt` 매개변수 타입 |
| `events-payload-mode.ts` | 모드(beyond/classic)별 이벤트 페이로드 타입 분리 |
| `nodes.ts` | `ServerExecutionFlowFunctions` — 380+ 노드 함수 메서드 시그니처 |
| `node_modes.ts` | `NODE_TYPE_BY_METHOD` — 각 함수의 beyond/classic 모드 지원 정보 |
| `zh_aliases.ts` | 중국어 → 영어 별칭 매핑 |

**주의:** 이 파일들을 직접 편집하면 다음 `npm run gen` 실행 시 덮어써진다. 반드시 `resources/node_definitions.json`을 수정하고 재생성한다.

## 수동 유지 파일

| 파일 | 내용 |
|------|------|
| `entity_helpers.ts` | 엔티티 서브타입 헬퍼 (`CharacterEntity`, `CreationEntity` 등), `ENTITY_HELPER_METHODS` |
| `prefabs.ts` | Prefab ID 상수 (`extractCustomResourcesFromGil`이 부분 업데이트 가능) |
| `server_on_overloads.d.ts` | `g.server().on()` 메서드 오버로드 타입 (이벤트별 `evt` 타입 분기) |
| `enum.ts` | 51개 게임 열거형 타입 (일부 자동 생성, 일부 수동) |

## 입력 데이터

### `resources/node_definitions.json`

게임 노드 시스템의 원시 데이터. 이벤트와 함수의 이름, 입/출력 핀 정의를 담는다.

### `resources/node_generics.json`

제네릭 타입 파라미터가 있는 함수들의 추가 정보. 예: `getListElement<T>`의 가능한 타입 목록.

## 코드 생성 과정

1. `node_definitions.json` 로드
2. 각 노드 정의 순회:
   - 이벤트 → `ServerEventMetadata` 항목 생성
   - 함수 → `ServerExecutionFlowFunctions` 메서드 시그니처 생성
3. 제네릭 노드는 각 가능한 타입에 대한 오버로드로 생성
4. 중국어 이름 매핑 추가
5. `prettier` 자동 포맷

## 타입 문자열 매핑

게임의 원시 타입 문자열을 genshin-ts 타입으로 변환하는 `TYPE_MAP`:

| 게임 타입 | genshin-ts 타입 |
|----------|----------------|
| `boolean` | `bool` |
| `integer` | `int` |
| `float` | `float` |
| `string` | `str` |
| `3d vector` | `vec3` |
| `entity` | `entity` |
| `prefab id` | `prefabId` |
| `dictionary` | `dict` |

## sendSignal 시그널 인자 지원 (포크 변경)

`src/definitions/nodes.ts`의 `sendSignal` 메서드에 `signalArgs` 매개변수가 추가되었다. 배열 타입의 인자는 `assemblyList` 노드로 자동 래핑된다.
