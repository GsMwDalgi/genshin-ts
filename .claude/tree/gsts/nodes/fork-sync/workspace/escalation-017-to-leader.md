# STATUS_REPORT (escalate) — Directive #017 — 방향 결정 요청

> From: fork-sync (distributor) → To: leader. Type: STATUS_REPORT.
> 이유: 미션 1축(우리 변형 유지) 예외 발동 = **방향(direction-level) 결정**. 나는 mode-blind라 독자 결정 금지 (distributor-role §You Do NOT). leader가 supervised/autonomous 적용해 택일 요청.
> impl 원본 escalation(증거 전체): `.claude/tree/gsts/nodes/fork-sync~impl/workspace/escalation-017.md`. merge log: `.../merge-log-017.md`.

## 한 줄
upstream `v0.1.10`(NEW_BASE `9cb31c8`, 71ca7a1서 43커밋 전진)이 우리 fork의 **signal-args 전체(A1~A13)와 동등한 기능을 독립 구현**했음(다른·더 정돈된 설계: `defineSignal()` + typed `evt.params`). 우리 명명(signalArgs/SignalArgDef 등) upstream에 0건 = upstream이 우리 코드를 가져간 게 아니라 평행 진화. → A그룹을 유지 재구현할지/폐기·채택할지/하이브리드할지가 미션 1축 방향 결정 → **택1 요청.**

## 가역성 (안전 — 아무것도 확정 안 됨)
- rebase abort 완료, src 워킹트리 무오염.
- `master` = `8629dc9` **불변**(force 안 함). 백업태그 `fork-backup-20260607` = `8629dc9`.
- `integ-upstream` 브랜치 존재(현재 == master). `upstream` remote 등록·fetch 완료(`upstream/master`=`9cb31c8`).

## 결정과 무관하게 확정된 사실 (모든 안 공통)
| 항목 | upstream 보유? | 처리 |
|------|----------------|------|
| **B1 varint guard** (signal_nodes.ts) | NO (grep 0) | 보존필요 — upstream signal_nodes 새 구조 위 재이식. §4 절대 롤백금지 유효. |
| **B2 any-guard** (ts_type_utils.ts) | NO | 보존필요(일반 타입추론 안전). A 폐기 시 효용 일부 감소 가능(검토). |
| **B5 entityLiteral parseValue** (nodes.ts) | NO | 보존필요 — signal-args와 독립(setCustomVariable entity 폴백). |
| **C CLI** (gil_inspect/gil_scaffold) | NO (신규파일) | 유지 자연스러움. gsts.ts 등록 24줄만 충돌(쉬움). upstream `gil_signals.ts`는 다른 파일·다른 목적(개념중첩, 코드충돌 아님). |
| **proto/gia_gen/thirdparty** | upstream 소유 | upstream 그대로 채택(우리 변경 0). |

## 결정 요청 — A그룹(signal-args) 처리 방향 택1

| 안 | 내용 | 장 | 단 |
|----|------|----|----|
| **A. Keep-both 재구현** | 우리 A1~A13을 upstream 새 구조 **위에** 재구현 | 미션1축 충실, 기존 사용자 코드 API 호환 | 동등기능 2개 공존(유지보수·네이밍/IR 충돌 위험), 미래 머지 마찰 지속 |
| **B. 폐기·upstream 채택** | A1~A13 폐기, upstream `defineSignal` 채택. B1/B2/B5/C만 유지 | upstream 추종(미션2축), 중복제거, 미래 머지 마찰↓ | 우리 API(`onSignal<Args>`/`SIGNAL_ARG_TYPE_MAP`) 사용처 마이그레이션. 동등성 review 확인 필수 |
| **C. 하이브리드** | upstream signal 코어 채택 + 우리 고유(예: _list 자동래핑·18종 맵)만 보강 | 중복 최소+우리 강점 보존 | 가장 복잡, 정밀 동등성 비교 선행 |

## 내 distributor 권고 (비구속)
- **B를 기본 권고, C를 차선.** 근거: upstream이 전 레이어 동등 + typed `evt.params`로 더 정돈된 설계 → 미션 2축(upstream 변형 최소화)·3축(통합상태 지속) 장기 부합. A는 동등기능 영구 중복이라 매 머지마다 같은 충돌 재발(미션 3축 역행).
- 단 **B/C 채택의 전제 = review의 독립 동등성 확인**(부록 기준): upstream defineSignal이 (1)18종 타입 커버리지, (2)`_list` 자동래핑 동등성을 충족하는지. 미충족 항목이 있으면 그 부분만 C로 보강(우리 기능 손실 방지). 이 동등성 검증을 **review 노드에 위임**할 것을 제안 — leader 승인 시 그렇게 분할.
- **autonomous 모드면**: leader가 B(또는 C) 지정 → 나는 impl STEP2를 "폐기·채택(+동등성 갭은 C보강)"으로 갱신 재dispatch, review가 동등성·B1/B2/B5·§5 독립확인. **supervised면**: 유저 확인 후 진행.

## 요청 사항 (leader에게)
1. A/B/C 택1.
2. (B/C 선택 시) 동등성 검증을 review 노드에 위임 승인 여부 — 미충족 갭은 C 보강으로 처리할지.
3. 마이그레이션 영향(B는 우리 기존 사용처 코드 변경 수반) 수용 여부.

## 현 상태
- impl: G0/G1 met(NEW_BASE 확정·게이트통과·브랜치/태그). G2~G5 blocked(방향대기, abort로 가역). 결정 후 재개.
- 나(fork-sync): impl/review 스펙·계약 작성 완료. 방향 확정되면 spec-impl-017 STEP2 갱신 → 재dispatch → review.
