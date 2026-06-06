# 05. 인젝터

## 개요

인젝터는 `.gia` 파일의 NodeGraph 바이트를 `.gil` 게임 맵 파일 내의 대상 NodeGraph와 교체한다. `.gil`은 프로토버프 기반 바이너리 포맷이며, 인젝터는 **전체 프로토버프 파싱을 피하고** 직접 varint 파싱으로 대상 NodeGraph의 바이트 범위만 찾아서 교체한다.

## 공개 API

**파일:** `src/injector/index.ts`

| 함수 | 설명 |
|------|------|
| `createInjector(protoPath?)` | Injector 객체 생성 |
| `injectGilFile(options)` | 파일 기반 주입 (읽기 → 패치 → 저장) |
| `injectGilBytes(input)` | 바이트 레벨 주입 핵심 함수 |

## .gil 바이너리 구조

```
Root message
  ├── Folder metadata (field 6)
  │     └── FolderIndex (field 1) × N   ← 폴더 인덱스 목록 (p0=6, p1=1)
  └── Content (field 10)
        └── NodeGraph blobs (field 10.1.1) × M  ← NodeGraph 원시 바이트 (p0=10, p1=1, p2=1)
```

폴더 메타데이터는 field 6 경로에, NodeGraph 블롭은 field 10 경로에 위치한다.

## 주입 흐름

1. `.gia` 바이트에서 NodeGraph ID 추출
2. `targetId` 결정 (명시값 > `.gia`에서 추론)
3. `findNodeGraphTargets()`로 `.gil` 내 교체 위치(바이트 오프셋) 파악
4. 안전 검사 실행
5. 필요시 `.gia`의 NodeGraph ID를 targetId로 재설정
6. 그래프 타입 패치 (불일치 시)
7. `applyReplacement()`로 바이너리 패치
8. 새 `.gil` 바이트 반환

## 안전 검사

| 검사 | 설명 |
|------|------|
| 빈 그래프 보호 | 대상 NodeGraph에 이미 노드가 있고 이름이 `_GSTS`로 시작하지 않으면 거부 |
| 그래프 타입 호환 | `.gia`와 `.gil` 대상의 그래프 타입(entity/status/class/item) 일치 검증 |
| 이름 접두사 | 새 NodeGraph 이름은 `_GSTS`로 시작해야 함 |

`skipNonEmptyCheck: true` 설정으로 우회 가능.

## 바이너리 파싱 모듈

**파일:** `src/injector/binary.ts`

| 함수 | 역할 |
|------|------|
| `readVarint(buf, offset)` | 프로토버프 varint 디코딩 |
| `encodeVarint(value)` | varint 인코딩 |
| `parseMessage(buf, ...)` | `.gil`을 재귀 순회하여 len-delimited 필드 위치 수집 (최대 depth=6, `ParseCollectors`로 10.1.1 블롭 별도 수집) |
| `applyReplacement(buf, patches)` | 바이트 배열에 패치 목록 적용 |

## 폴더/인덱스 구조

**파일:** `src/injector/folder.ts`

`.gil`의 폴더 메타데이터를 파싱한다. 각 FolderEntry는 그래프 타입값(`typeValue`)과 NodeGraph ID를 포함한다.

### 이중 경로 탐색 (Dual-Path Resolution)

`findFolderEntryField()`는 대상 ID를 두 경로에서 탐색한다:
- Content 엔트리: path `6.1.3.5` (depth=4)
- Meta 엔트리: path `6.1.2.4.5` (depth=5)

다수 매치 시 Content 엔트리를 우선한다.

### 타입 해석 폴백 체인 (Type Resolution Fallback)

`resolveGraphTypeForTypeValue()`는 다단계 폴백으로 그래프 타입을 결정한다:
1. 같은 typeValue를 가진 기존 그래프들의 실제 `NodeGraph.Id.type`에서 추론
2. 단일 값이면 채택; 복수 값이면 충돌 에러
3. 매핑 없으면 `DEFAULT_VALUE_TO_GRAPH_TYPE` 정적 맵으로 폴백

`.gil` 파일 내 `typeValue` 필드와 코드에서 사용하는 그래프 타입 ID는 다른 값이다. `DEFAULT_GRAPH_TYPE_VALUES` 매핑으로 변환한다:

| typeValue (.gil) | 그래프 타입 ID (코드) | 그래프 타입 |
|-----------------|---------------------|------------|
| 800 | 20000 | entity |
| 2300 | 20003 | status |
| 2400 | 20004 | class |
| 4300 | 20005 | item |

## 시그널 노드 ID 패치

**파일:** `src/injector/signal_nodes.ts`

`patchSignalNodeIds()`가 `onSignal`/`sendSignal` 이벤트 노드의 시그널 파라미터 ID를 올바르게 패치한다.

### sendServer 시그널 종류

시그널 노드 종류 판별 시 `type=20002` (`SIGNAL_NODE_TYPE_SKILLS`)이면 `sendServer`로 분류된다. 이 값은 `signal_nodes.ts:26`에 하드코딩되어 있으며 Skills 그래프 타입의 시그널 전송을 나타낸다.

### 시그널 노드 파싱 개선 (포크 변경)

1. **varint 오버플로우 방지**: 모든 파싱 루프에 오프셋 음수/오버플로우 검사 추가
2. **ClientExec 핀 우선 탐색**: 시그널 이름 추출 시 InParam 문자열 값과 혼동을 방지하기 위해 ClientExec 핀(kind=5)을 먼저 검색

## GIL Reader (포크 추가)

**파일:** `src/injector/reader.ts`

`.gil` 바이너리에서 NodeGraph 상세 정보를 읽는 범용 리더. `inspect`과 `scaffold` CLI 명령의 기반이 된다.

| 함수 | 반환 타입 | 설명 |
|------|----------|------|
| `readGilNodeGraphs(gilBytes)` | `NodeGraphSummary[]` | 모든 NodeGraph 요약 목록 |
| `readGilNodeGraph(gilBytes, id)` | `NodeGraphDetail` | 특정 NodeGraph 상세 (변수, 노드, 연결) |

## CLI에서의 주입 흐름

`src/cli/gsts.ts`의 `maybeInjectGia()`:

1. `resolveGilFolder()` — 게임 설치 경로에서 BeyondLocal 폴더 찾기
2. `resolveGilTarget()` — `<playerId>/<mapId>.gil` 경로 결정
3. 백업 생성 — `<backupsDir>/<playerId>/<mapId>/<timestamp>.gil`
4. `injectGilFile()` 호출
5. 성공/실패 UI 출력

## Proto 스키마 로딩/캐싱

**파일:** `src/injector/proto.ts`

`gia.proto` 스키마를 `protobufjs`로 로드한 뒤 절대경로 기준으로 `Map`에 캐싱한다. `Root`와 `NodeGraph` 메시지 타입을 재사용한다. 스키마가 로드 시점에 고정되므로 게임 버전 변경에 런타임 대응 불가.

## 헤더 검증 동작

인젝터는 `head_tag`(0x0326)과 `tail_tag`(0x0679)만 검증한다. `file_type`과 `schema_version`은 읽어서 보존하지만 **검증하지 않는다** — 잘못된 값이 들어와도 에러 없이 통과한다.

## 주의 사항

1. **맵 에디터에서 저장 후에만 NodeGraph 항목이 생성된다.** 에디터에서 NodeGraph를 만들고 저장하지 않으면 인젝터가 대상을 찾지 못한다.
2. **전체 프로토버프 파싱을 피한다.** `protobufjs`로 전체 `.gil`을 파싱하면 매우 느리다.
3. **NodeGraph 블롭 위치는 `depth=3, p0=10, p1=1, p2=1` 경로에 고정되어 있다.**
4. **`parseMessage` 재귀 깊이는 6으로 제한된다.** depth >= 6인 중첩 필드는 인덱싱되지 않는다.
