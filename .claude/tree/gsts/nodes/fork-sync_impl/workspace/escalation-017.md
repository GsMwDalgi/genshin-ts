# ESCALATION — Task #017 — upstream이 signal-args(A그룹)를 자체 도입

> From: fork-sync~impl → To: fork-sync (parent). Type: STATUS_REPORT (escalate).
> 트리거: spec-impl-017.md line 71-72 "upstream이 signal-args를 자체 도입해 우리 전체 폐기가 합리적으로 보일 때 (방향성 결정 — 폐기는 독자 금지)".
> 계약 G5 / spec STEP2 line 36: 변형 폐기는 **독자 금지**, review 독립확인 대상. → 방향 결정을 parent에 요청.

## 한 줄 요약
upstream `v0.1.10`(NEW_BASE `9cb31c8`)이 우리 fork의 **signal-args 기능 전체(A1~A13)** 와 **동등한 네이티브 시스템을 독립 구현**했음. 어느 방향으로 통합할지(우리 A그룹 유지 재구현 / 폐기·upstream 채택 / 하이브리드)는 미션 1축 방향 결정이라 독자 판단 금지 → **parent 결정 요청**. B1/B2/B5 버그수정과 C CLI 도구는 별개(아래).

## 증거 — upstream 네이티브 signal-args (전 레이어 동등)

| 레이어 | 우리 fork (A그룹) | upstream v0.1.10 | 판정 |
|--------|-------------------|------------------|------|
| Runtime 저작 | `onSignal<Args extends readonly SignalArgDef[]>`, `sendSignal(...signalArgs)` (A2/A8) | `defineSignal()` → `SignalDefinition`, `onSignal<S extends SignalDefinition>` typed `evt.params`, `isSignalDefinition` (core.ts L74-152) | **동등 기능, 다른 설계** |
| Runtime 타입 | `SignalArgDef`, `SignalArgsToPayload` (value.ts, A1) | `SignalParamType`, `SignalParamEntry/Entries`, `SignalParamValues`, `SignalEventPayload` (core.ts L48-118) | **동등** |
| Compiler IR→GIA | `SIGNAL_ARG_TYPE_MAP` 18종 (mappings.ts A10), send/monitor 핀 생성 (index.ts A11) | `send_signal:300000`/`monitor_signal:300001` (mappings L421-422) + per-param 핀 생성 루프 (index.ts L324-356) | **동등** |
| Injector | ClientExec 우선탐색 (signal_nodes.ts A13) | upstream signal_nodes.ts 자체 92줄 수정 | 구조 상이(조사 필요) |
| 추출 CLI | (우리 C: gil_inspect/gil_scaffold) | `gil_signals.ts` `extractSignalsFromGil` + inject시 자동추출 (gsts.ts) | 개념 중첩 |

검증 커맨드(재현 가능):
- `git show upstream/master:src/runtime/core.ts | grep -nE "SignalDefinition|defineSignal|onSignal"` → L74,128,345 등.
- `git show upstream/master:src/compiler/ir_to_gia_transform/index.ts | grep -nE "send_signal|signalParams"` → L324,338,356.
- `git show upstream/master:src/compiler/ir_to_gia_transform/mappings.ts | grep send_signal` → L421 `send_signal: 300000`.
- `git grep -nE "signalArgs|SignalArgsToPayload|SignalArgDef" upstream/master -- src/` → **0건** (우리 명명은 upstream에 전무 — 즉 upstream은 우리 코드를 안 가져갔고 독립 구현).

## 버그수정 B1/B2/B5 — upstream **미보유** (어느 방향이든 보존 필요)
| 버그fix | upstream 보유? | 결론 |
|---------|----------------|------|
| B1 varint guard (signal_nodes.ts) | **NO** (grep 0건) | 보존필요 — A그룹 폐기해도 signal_nodes 무한루프 가드는 살려야. upstream PR 역제안 후보. |
| B2 any-guard (ts_type_utils.ts) | **NO** | 보존필요 (A 폐기 시에도 일반 타입추론 안전장치) — 단 A 폐기되면 B2 효용은 줄 수 있음(검토). |
| B5 entityLiteral parseValue (nodes.ts) | **NO** | 보존필요 — setCustomVariable entity 폴백, signal-args와 독립. |

→ §4 "절대 롤백금지"는 방향과 무관하게 유효. 단 B1은 upstream signal_nodes 새 구조 위 재이식 필요(구조 충돌).

## C CLI 도구 (gil_inspect / gil_scaffold) — 충돌 경미, 별도 판단 가능
- 두 파일 모두 upstream에 **부재**(신규파일) → 파일충돌 0. `gsts.ts` 서브커맨드 등록 24줄만 충돌(upstream도 gsts.ts를 47줄 수정 = commander `.command()` 스타일).
- upstream `gil_signals.ts`는 우리 도구와 다른 파일·다른 목적(inject시 시그널 자동추출 vs 우리 대화형 inspect/scaffold). **개념 중첩이나 코드 충돌 아님.** C는 유지가 자연스러움(미션 1축, 우리 독자 도구).

## 결정 요청 (3안)
| 안 | 내용 | 장 | 단 |
|----|------|----|----|
| **A. Keep-both 재구현** | 우리 A그룹을 upstream 새 구조 **위에** 재구현(spec 기본방침) | 미션1축 충실, 기존 사용자 코드 호환 | upstream과 중복 기능 2개 공존, 유지보수 부담↑, 네이밍/IR 충돌 위험 |
| **B. A그룹 폐기·upstream 채택** | signal-args A1~A13 폐기, upstream defineSignal 채택. B1/B2/B5·C만 유지 | upstream 추종(미션2축↑), 중복제거, 미래 머지 마찰↓ | 우리 API(onSignal<Args>/SIGNAL_ARG_TYPE_MAP) 사용처 마이그레이션 필요. 18종 커버리지/_list 자동래핑 동등성 review 확인 필수(부록 기준) |
| **C. 하이브리드** | upstream signal 코어 채택 + 우리 고유 기능(예: _list 자동래핑, 18종 맵)만 그 위 보강 | 중복 최소+우리 강점 보존 | 가장 복잡, 동등성 정밀비교 선행 필요 |

impl 의견(잠정, 비구속): spec 기본은 A이나, upstream이 **전 레이어 동등 + 더 정돈된 typed `evt.params` 설계**라 미션 3축(통합상태 지속/upstream 마찰 최소) 관점에서 **B 또는 C**가 장기적으로 합리적으로 보임. 단 이는 방향 결정이라 **독자 진행 안 함**. parent가 A/B/C 택1 + (B/C라면)동등성 기준을 review에 위임할지 지정 요청.

## 진행 차단/비차단 구분
- **차단(결정 대기)**: STEP2 HIGH-7 충돌해소의 signal-args 부분(core/nodes/index/signal_nodes의 A기능 재구현 vs 폐기) — 방향 미정이면 진행 불가.
- **결정과 무관하게 가능(요청 시 선행)**: proto/gia_gen upstream 채택(STEP3), B1/B2/B5 재이식, C tools gsts.ts 등록 충돌해소, npm install/빌드 환경 확인. parent가 "B1/B2/B5+C+proto만 선행" 지시 시 부분 진행 가능.

## 현 상태 (가역성 확인)
- rebase **abort 완료** → 워킹트리 src 무오염.
- `master` = `8629dc9` 불변. 백업태그 `fork-backup-20260607` = `8629dc9`. `integ-upstream` 브랜치 존재(=master).
- `upstream` remote 등록·fetch 완료(`upstream/master` = `9cb31c8` = v0.1.10).
- 상세 진행로그: `workspace/merge-log-017.md`.

## self-eval (escalate 시점)
- G0: **met** — NEW_BASE=9cb31c8 확정, 게이트 통과(진행 판정), 위상분석 완료.
- G1: **met** — 백업태그·작업브랜치 생성, master 불변 확인.
- G2~G5: **blocked** — 방향 결정 대기로 미수행. abort로 가역 보존. parent 결정 후 재개.
