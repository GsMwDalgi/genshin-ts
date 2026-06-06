# 포크 변경사항 완전 목록 (Fork Changes Inventory)

> **작성일:** 2026-06-06 / **Directive #012**
> **목적:** upstream(josStorer/genshin-ts) 최신본 병합 전, 우리 포크가 원본 대비 가한 **모든 커스텀 변경**을 카테고리별로 빠짐없이 정리. 병합 시 충돌 우선 확인 지점 식별 + 유저 기억 회복용 기준 자료.

---

## 검증 방법 (Methodology)

| 항목 | 내용 | 확신도 |
|------|------|--------|
| 포크 분기점 | `71ca7a1 update proto for client nodes` (upstream tip). 우리 커밋은 그 위에 **선형(linear)** 으로 적재됨 — 분기/머지 없음 | **확인** (git log --graph) |
| 정확한 delta | `git diff 71ca7a1 HEAD` = 84 files, +10604/-27. 이것이 포크 변경의 **완전한 권위 소스** | **확인** |
| 교차 검증 | `notes/changelog.md`(#1~9), 실제 src 코드 diff, `.claude/handoff-gsts.md` Decisions, git log 대조 | **확인** |

> **changelog vs git 불일치 (경미):**
> 1. changelog는 baseline을 "v0.1.7(`6f288e9`)"로 적었으나, 실제 분기점은 그 다음 upstream 커밋 `71ca7a1`. → `71ca7a1`의 proto 변경(client nodes)은 **upstream 것**이지 우리 변경이 아님. 병합 시 혼동 주의.
> 2. changelog 항목 번호 #7(.gitignore)이 #9 뒤에 배치됨(순서만 어긋남, 내용 누락 없음).
> 3. **src 코드 변경은 changelog #1~9와 git diff가 100% 일치** — 문서에 안 적힌 숨은 코드 변경 없음(확인).

> **추측 vs 확인:** 아래 "변경 내용/이유/위험도"는 git diff + 코드 + changelog로 **확인**된 사실. 병합 처리 방향 메모만 일부 **추론(inferred)** 이며 그렇게 표기함.

---

## A. 신규 기능 — 시그널 커스텀 인자 (Signal Arguments)

가장 크고 위험한 변경. `sendSignal`/`onSignal`/`send()`에 최대 18종 커스텀 인자(entity, int, str, vec3, *_list 등)를 실어 보낼 수 있게 함. **런타임·컴파일러·인젝터·정의 4개 레이어를 관통**.

| # | 파일 (경로) | 변경 내용 | 이유 | 위험도 |
|---|------------|----------|------|--------|
| A1 | `src/runtime/value.ts` | `SignalArgDef`, `SignalArgsToPayload` 타입 추가 (+22) | 시그널 인자 정의/페이로드 타입 유도 | **MED** |
| A2 | `src/runtime/core.ts` | `onSignal<Args>` 제네릭화, `registerEvent`/`runServerHandler`/`runHandler`에 `signalArgs` 전파, 인자별 output pin 생성 (+68/-7) | 핸들러가 인자 타입 수용·전파 | **HIGH** |
| A3 | `src/runtime/ir_builder.ts` | `buildSignalNode` 추가, `send_signal`/`monitor_signal`에 `signalParams` 포함 (+10) | IR에 시그널 파라미터 보존 | **MED** |
| A4 | `src/runtime/meta_call_types.ts` | `MetaCallRecord.signalParams` 필드 추가 (+1) | 메타콜 레코드에 파라미터 | **LOW** |
| A5 | `src/runtime/IR.d.ts` | `Node.signalParams` 필드 추가 (+1) | IR 타입에 필드 | **LOW** |
| A6 | `src/runtime/server_globals.ts` | `send()`에 `args?` 매개변수 추가 (+4/-4 영역) | 전역 `send()` 인자 전달 | **MED** |
| A7 | `src/runtime/server_globals.d.ts` | `send(name, args?: Record<string,any>)` 시그니처 (+1/-1) | 타입 선언 | **LOW** |
| A8 | `src/definitions/nodes.ts` | `sendSignal`에 `signalArgs` 배열 매개변수 + 배열 자동 래핑(assemblyList) (+29 영역) | 인자 직렬화 + _list 처리 | **HIGH** |
| A9 | `src/definitions/events-payload.ts` | `value` import 추가 (+1) | 페이로드 타입용 | **LOW** |
| A10 | `src/compiler/ir_to_gia_transform/mappings.ts` | `SIGNAL_ARG_TYPE_MAP` (18종 typeId/dataGroup/elementType 매핑) 추가 (+24) | GIA 핀 타입 인코딩 | **HIGH** |
| A11 | `src/compiler/ir_to_gia_transform/index.ts` | `send_signal` 입력핀 / `monitor_signal` 출력핀 생성, `send_signal` 인덱스 매핑(`idx-1`) (+33/-1) | GIA 변환에 핀 생성 | **HIGH** |
| A12 | `src/compiler/ir_to_gia_transform/pins.ts` | `ensureInputPinWithType()` 추가, `send_signal` entity null 핀 타입 처리 (+21) | conn 타입 핀 인코딩 매칭 | **HIGH** |
| A13 | `src/injector/signal_nodes.ts` | `extractSignalNameFromNode`에 ClientExec 핀(kind=5) 우선 탐색 (+pin i1/i2.kind 타입) | InParam 문자열 오매칭 방지 | **HIGH** |

**병합 처리 방향(추론):** 이 기능은 upstream에 없는 독자 기능. upstream이 동일 기능을 도입하면 우리 구현을 버리고 채택 검토. 미도입 시, 위 HIGH 항목들은 병합 후 **반드시 재적용·재검증** 필요(특히 A2/A8/A10~A13). signal_nodes.ts/index.ts/pins.ts는 upstream도 자주 건드리는 핫스팟이라 충돌 가능성 높음.

---

## B. 버그 수정 (Bug Fixes)

| # | 파일 | 변경 내용 | 이유 | 위험도 | 병합 메모(추론) |
|---|------|----------|------|--------|------|
| B1 | `src/injector/signal_nodes.ts` | varint 파서 3함수(`readFieldMessages`/`readFieldBytes`/`parseNodeGraphId`)에 오프셋 음수·오버플로우 가드 추가 (`offset>=0` 조건, `len<0`/`dataEnd<0`/`newOffset<0` break) | 32bit 정수 오버플로우로 인젝션 무한루프 | **MED** | upstream 미수정 시 재적용. 방어적 가드라 upstream에 PR 가능한 성격 |
| B2 | `src/shared/ts_type_utils.ts` | `isEntityLikeType` 시작부 `if (type.flags & ts.TypeFlags.Any) return false` (+2) | `any`가 entity로 오추론(`isTypeAssignableTo(any,entity)===true`) → 중간 변수 시그널 인자 오추론 | **MED** | A 기능 의존적. 문제 시 이 1줄 삭제로 롤백(인자를 인라인 사용) |
| B3 | `src/runtime/core.ts` (A2에 포함) | `onSignal`이 `SignalArgDef[]` 수용·전파 못해 GIA 생성 실패 수정 | changelog #8 | **HIGH** | A2와 동일 위치 |
| B4 | `src/compiler/ir_to_gia_transform/index.ts` (A11에 포함) | `monitor_signal` output pin 생성 로직 | changelog #8 | **HIGH** | A11과 동일 위치 |
| B5 | `src/definitions/nodes.ts` `parseValue` | `entity` case에 `entityLiteral` bigint/int 폴백 추가 (+entityLiteral import) | `setCustomVariable`에 entity 값 전달 시 `Invalid value type: entity` | **MED** | changelog #8. parseValue는 핫스팟, 충돌 주의 |

> B3/B4는 changelog #8(버그수정)으로 분류되나 코드상 A2/A11과 같은 hunk에 섞여 있음 — 병합 시 분리 불가, A 기능과 함께 취급.

---

## C. 신규 기능 — 개발 도구 (CLI / Reader)

| # | 파일 | 변경 내용 | 이유 | 위험도 |
|---|------|----------|------|--------|
| C1 | `src/cli/gil_inspect.ts` | **신규 파일** (+153). `gsts inspect <file> [--id --json --raw]` | .gil NodeGraph 조회 | **LOW** |
| C2 | `src/cli/gil_scaffold.ts` | **신규 파일** (+236). `gsts scaffold <file> --id [--out --force]` | .gil→TS 스캐폴드 생성 | **LOW** |
| C3 | `src/injector/reader.ts` | **신규 파일** (+367). `readGilNodeGraphs`/`readGilNodeGraph` 범용 리더 (inspect/scaffold 공통 기반) | GIL NodeGraph 상세 파싱 | **LOW** |
| C4 | `src/cli/gsts.ts` | `inspect`/`scaffold` 서브명령 등록 + import (+24) | CLI 진입점 연결 | **MED** |
| C5 | `src/i18n/locales/en-US/main.json` | inspect/scaffold i18n 문자열 10개 추가 | CLI 다국어 | **LOW** |
| C6 | `src/i18n/locales/zh-CN/main.json` | inspect/scaffold i18n 문자열 10개 추가 | CLI 다국어 | **LOW** |

**병합 메모(추론):** C1~C3는 신규 파일이라 충돌 없음(LOW). C4(gsts.ts)와 C5/C6(i18n json)은 **기존 파일에 추가**이므로 upstream이 같은 파일에 명령/문자열을 추가했다면 충돌 가능 → 추가 위치만 조정하면 됨(MED, 쉬움). 기능 자체는 우리 독자 도구이므로 그대로 유지.

---

## D. 설정 (Config)

| # | 파일 | 변경 내용 | 이유 | 위험도 |
|---|------|----------|------|--------|
| D1 | `.gitignore` | `.vs/`, `.claude/` 제외 추가 (+4/-2 영역) | Visual Studio / Claude Code 산출물 제외. 단, 일부 `.claude/`는 의도적으로 트래킹(아래 E 참조) | **LOW** |

---

## E. 문서 / 프로젝트 자산 (무위험, 코드 영향 0)

코드 컴파일/런타임에 영향 없음. upstream `docs/`는 건드리지 않고 전부 `notes/`·`.claude/`에 분리 → **병합 충돌 위험 없음(LOW)**.

| # | 위치 | 내용 |
|---|------|------|
| E1 | `notes/changelog.md` | 포크 변경 로그(#1~9) (+227) |
| E2 | `notes/protocol/` (21개 파일) | GIA/GIL 프로토콜 분석 통합 문서 (changelog #9: 두 소스 18문서로 병합, CONFIRMED/INFERRED/SPECULATED 분류) |
| E3 | `notes/architecture/` (8개 파일, 01~08) | 아키텍처 문서 |
| E4 | `.claude/` (handoff, skills/node-graph-coder, prompt-reference, tree agent 상태, directives, settings.local.json) | 트리 에이전트 상태·스킬·프로젝트 컨텍스트 (의도적 git 트래킹 — D1과 상충하지만 명시적 add됨) |

---

## F. 검증으로 확인된 "변경 아님" (병합 혼동 방지)

| 항목 | 사실 | 근거 |
|------|------|------|
| **proto / protobuf 스키마** | 우리 fork의 git diff에 `*.proto`·`gia_gen`·`thirdparty` 변경 **0건** | `git diff 71ca7a1 HEAD --stat` 결과 없음 |
| handoff "protobuf 차이"(Decision #55) | `71ca7a1 update proto for client nodes`는 **upstream** 작업 + notes의 역분석 분석 결과. 우리가 .proto를 수정한 게 아님 | git log(71ca7a1는 분기점), diff 부재 |

> ⚠️ 병합 시 주의: handoff/notes의 "신규 노드 타입(CreationStatusDecision/Skill/Status), 필드명 확정, Dictionary, ClientSignal(kind=6)" 서술은 **분석 문서**이지 우리 코드 변경이 아님. proto는 upstream 소유 → 최신 upstream proto를 그대로 채택하면 됨.

---

## 요약 표 (Summary)

### 카테고리별 항목 수

| 카테고리 | 항목 수 | 비고 |
|---------|--------|------|
| A. 신규기능 — 시그널 인자 | 13 (코드 파일) | 단일 기능, 4개 레이어 관통 |
| B. 버그 수정 | 5 (B3/B4는 A와 동일 hunk) | 실질 독립 수정 3건(B1/B2/B5) |
| C. 신규기능 — CLI/Reader | 6 (신규 3 + 수정 3) | |
| D. 설정 | 1 | |
| E. 문서/자산 | ~38 파일 | 코드 영향 0 |
| **합계 (코드 영향 파일)** | **22 src 파일** | + 1 `.gitignore` |

### 충돌 위험 분포 (코드 변경 한정)

| 위험도 | 항목 | 병합 시 우선순위 |
|--------|------|-----------------|
| **HIGH** | A2(core.ts), A8(nodes.ts sendSignal), A10(mappings.ts), A11(index.ts), A12(pins.ts), A13/B(signal_nodes.ts), B5(nodes.ts parseValue) | **최우선 재검증.** 모두 시그널 인자 기능 + 컴파일러/인젝터 핫스팟 |
| **MED** | A1, A3, A6, B1, B2, C4, D1 | 추가/시그니처 변경 — 충돌 가능하나 해결 쉬움 |
| **LOW** | A4, A5, A7, A9, C1~C3, C5, C6 | 신규 파일·단순 필드 추가 |
| **무위험** | E 전체(notes/.claude) | upstream `docs/` 미접촉 |

### 병합 핵심 권고

1. **HIGH 7개 지점**(core.ts·nodes.ts·mappings.ts·index.ts·pins.ts·signal_nodes.ts)이 충돌의 90%. 시그널 인자 기능을 하나의 패치 세트로 묶어 최신 upstream 위에 재적용·재검증.
2. **proto는 upstream 채택** — 우리 변경 없으므로 최신본 그대로 사용.
3. **C(CLI)·E(문서)는 안전** — 신규 파일/분리 디렉터리라 거의 충돌 없음.
4. **B1·B2 방어 가드**는 upstream에 역제안(PR) 가능한 일반 버그 수정.
