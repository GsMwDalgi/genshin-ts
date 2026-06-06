# Memo

## ✅ #017 종결 (2026-06-07 gen3 재개) — UPSTREAM MERGE 방향B PASS, leader 보고완료
- **재개 완료**: startup 재실행 → review TASK_RESULT(audit `notes/integration/upstream-merge-audit.md`) + impl result(`fork-sync_impl/workspace/result-017.md`) 수령. **자식 디스크명 = `_`** (fork-sync_impl / fork-sync_review). impl result 경로는 옛 `~`로 적혀있으나 실파일=`fork-sync_impl`.
- **review VERDICT = PASS, blocking 0.** R1~R6 전부 독립 재실행 PASS. R5(폐기 동등성 게이트) = 폐기 확정 PASS(갭없음, upstream⊇우리, C-하이브리드 불요).
- **fork-sync 독립 재확인 (디스크 직접, 전부 일치)**: master=backup=8629dc9(force0) / upstream/master=9cb31c8 / HEAD=integ-upstream / 71ca7a1=ancestor(exit0) / A잔재 git grep=0 / diff=10파일 +832/-6 B·C만 / B1=signal_nodes2+binary7=9·B2=1·B5=1 / **R5 집합대조 직접**: 옛18종(backup tag mappings.ts L405-423) ⊆ upstream SignalParamType(core.ts L48-69, 21항=18+faction/faction_list/unknown), ours−upstream=∅ → superset 확정.
- **verdict 작성 완료**: `workspace/verdict-017.md`. leader TASK_RESULT 보고완료.
- **gap 없음 → impl 재활성화(C-patch) 불요.** non-blocking 2건만(signal e2e fixture 깊이검사 미완주=우리무관·leader수용시 보강 / R6 sendSignal 라인번호 5줄 드리프트=경미).
- **master force·커밋은 leader 소관**(review PASS+§5+leader 수용 후). 내가 안 함.
- **다음**: leader 수용/추가지시 대기. 새 fork-maintenance 작업 시 fork-sync_impl/fork-sync_review 재사용.

## ⚠️ 노드명 구분자 변경 (2026-06-07 ~02:5x, leader 스킬업데이트)
- **런타임이 `~` 거부 → 자식 노드명 `~` → `_` 마이그레이션됨**: `fork-sync~impl` → **`fork-sync_impl`**, `fork-sync~review` → **`fork-sync_review`** (디스크 rename). 아래 메모/스펙의 `fork-sync~impl/~review` 표기는 전부 **`fork-sync_impl`/`fork-sync_review`로 읽을 것**. tree-cli send / SendMessage 시 `_` 이름 사용.
- **leader가 review의 TASK_RESULT를 재전달 예정**(`~` 라우팅 깨져서). 재활성화 직후 그 review 결과로 #017 verdict 진행.
- **재개 절차**: ① startup fork-sync 재실행 ② review TASK_RESULT(audit `notes/integration/upstream-merge-audit.md`) 수령 확인 ③ R5 동등성 게이트 판정 채택 ④ verdict-017 작성 ⑤ leader 보고. (impl 단계는 이미 완료·독립확인 끝 — 아래 ★ 02:3x 참조.)

## ▶ RESUME BOOKMARK (2026-06-07 02:0x, #017 진행 중)
**나 = fork-sync (distributor), parent=leader. 역할: fork 유지보수 총괄, impl(fork-sync_impl)·review(fork-sync_review) 분할 조율. [구.~ → 현._]**
- **현재 작업 = Directive #017 — UPSTREAM MERGE 실제 실행.** upstream URL 확보됨: `https://github.com/josStorer/genshin-ts`.
  - Directive: `.claude/tree/gsts/directives/017-upstream-merge.md`. Mode=autonomous, model=opus.
  - 절차 = `notes/integration-workflow.md` §1~§5 (권위 절차). 분류참조 = `notes/fork-changes-inventory.md`. 번들 = `notes/integration/patches/` (20패치).
- **핵심 제약**: ① 전용 브랜치(`integ-upstream`)+백업태그(`fork-backup-YYYYMMDD HEAD`) 먼저, master force 금지(검증+수용 전까지). ② NEW_BASE==71ca7a1이면 머지할 것 없음→보고/중단. ③ B1/B2/B5 절대 롤백금지(§4 grep 독립검증). ④ proto/gia_gen/thirdparty=upstream 그대로. ⑤ impl≠review. ⑥ 루트 clean, temp worktree/R: 사용. ⑦ 변형폐기는 review 독립확인 후만(부록 기준).
- **git 현황(dispatch 시점)**: HEAD=`8629dc9`(leader가 #014~016 베이스라인 커밋함, memo 옛값 594a6e8 아님). 분기점=71ca7a1. upstream remote 미등록(impl이 add). 워킹트리 code-clean.
  - ⚠️ workflow §1의 "우리 커밋 66f4803,9415773"는 옛 예시 — impl은 실제 `git log 71ca7a1..HEAD`로 커밋레인지 판단할 것(하드코딩 금지).
- **분할 결정**: 단일 머지 시퀀스. impl이 §1~§5 전부 실행(브랜치+태그+rebase/패치재적용+충돌해소 HIGH-7+검증). review가 §4(B1/B2/B5 grep)+§5(install/tsc/build/smoke) 독립 재실행 + 변형폐기 결정 독립확인. master force는 안 함(검증+leader 수용 후 별도).
- **진행**: [x] contract-017 (`workspace/contract-017-upstream-merge.md`) [x] impl 스펙 (`workspace/spec-impl-017.md`) [x] impl seed (unread.20260607T0201) + leader 활성화 SPAWN_REQUEST 송부 [ ] impl 수령+독립확인 [x] review 스펙 (`workspace/spec-review-017.md`) [ ] review seed+활성화(impl 종결 후) [ ] review 수령 [ ] verdict-017 [ ] leader 보고.
- **★★ 02:1x impl ESCALATE 수령 → leader 방향결정 요청 송부 (현 차단상태)**:
  - upstream v0.1.10(NEW_BASE=`9cb31c8`, 71ca7a1서 +43커밋)이 **signal-args 전체(A1~A13) 동등 기능을 독립 구현**(다른·더 정돈된 설계: `defineSignal()`+typed `evt.params`). upstream에 우리 명명(signalArgs/SignalArgDef) 0건=평행진화. → 미션1축 예외=**방향결정**. 나=mode-blind라 독자결정 금지(distributor-role).
  - **가역성 확인됨**: rebase abort 완료, master=`8629dc9` 불변, 백업태그 `fork-backup-20260607`=8629dc9, `integ-upstream` 브랜치(=master), upstream remote 등록(`upstream/master`=9cb31c8). src 무오염.
  - **모든 안 공통(확정)**: B1/B2/B5=upstream 미보유(grep0)→보존필요(B1은 upstream signal_nodes 새구조 위 재이식). C CLI=upstream 부재 신규파일→유지(gsts.ts 24줄만 충돌). proto=upstream 채택.
  - **3안**: A=keep-both 재구현 / B=A폐기·upstream채택(+B1/B2/B5/C유지) / C=하이브리드. impl 잠정 B/C. 내 권고=**B기본,C차선**(미션2/3축). B/C 전제=review 독립 동등성확인(18종+_list 자동래핑); 갭은 C보강.
  - 파일: 내 보고 `workspace/escalation-017-to-leader.md`, impl 원본 `.claude/tree/gsts/nodes/fork-sync~impl/workspace/escalation-017.md`, log `.../merge-log-017.md`.
  - **STATUS_REPORT/SPAWN_REQUEST는 tree-cli send 불가**(persistent type 아님: TASK_ASSIGN/TASK_RESULT만) → SendMessage로 leader 직접. (이번에 재확인.)
  - (해소됨 — 아래 참조.)

- **★★★ 02:2x leader 방향결정 수령 = B 확정 (autonomous + user-confirmed)**:
  - directive `017-upstream-merge.md` "## Leader Decision" 추가됨. **방향 B = group A(A1~A13) 폐기, upstream `defineSignal`/typed `evt.params` 채택.**
  - leader 추가 요구: ① B/C 동등성검증 review 위임 **승인**(갭→C-하이브리드 보강). ② **마이그레이션 델타 문서화 필수**(우리 옛 API→upstream 등가, verdict가 leader에 전달, downstream gsts-sandbox/팀git/mwe용). ③ 마이그레이션 영향 **수용**. ④ master force 금지(review PASS+§5+leader 수용 후 별도).
  - **spec 갱신 완료**:
    - `spec-impl-017.md` = **REV2**: STEP0/1 사전작업 완료라 재확인만, STEP2="A 폐기+upstream 채택+B/C 재이식"(파일별 처리표), B5/B1은 A8B5·A13B1 패치서 B hunk만 추출, result에 마이그레이션 델타 표 필수, G5=폐기실행+동등성갭 관찰.
    - `spec-review-017.md`: R2=방향B 정합성(A잔재0 확인+upstream채택+B/C재이식), R5=★폐기 동등성 게이트(18종+_list, 미검증폐기=치명FAIL, 갭=C보강), R6=마이그레이션 델타 정확성.
  - **impl 재dispatch 완료**: tree-cli send TASK_ASSIGN (unread.20260607T0220) → **SEND(노드 ALIVE)** → SendMessage로 wake 완료. (escalate 후에도 살아있어서 SPAWN_REQUEST 불요였음.)
  - (impl REV2 완료 — 아래.)

- **★ 02:3x impl TASK_RESULT 수령 + fork-sync 독립확인 완료**:
  - result: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-017.md` (log `merge-log-017.md`). 방식=`git reset --hard upstream/master` + B1/B2/B5+C만 수동 재이식(group A 0줄 재적용). G0~G4 met, G5=방향B실행+superset관찰(폐기확정 review대기).
  - **fork-sync 디스크 독립확인 (전부 일치 ✔)**:
    - master=`8629dc9` 불변 / 백업태그=8629dc9 / upstream/master=9cb31c8 / 현브랜치=integ-upstream.
    - **A잔재=0** (`git grep -nE "signalArgs|SignalArgsToPayload|SignalArgDef|SIGNAL_ARG_TYPE_MAP" -- src/` = 0건).
    - B1 분산: signal_nodes.ts(L36,L56)=2 + binary.ts(L44/85/54/95/57/98 +L194 upstream기존)=7 → 3함수 가드 보존. **upstream이 readField*를 signal_nodes→binary.ts 이동한 결과**(가드누락 아님 — §4 grep을 signal_nodes 단독으로 보면 2건뿐이라 오인 주의, binary.ts 합산필수).
    - B2=1(ts_type_utils.ts:28), B5=1(nodes.ts:250).
    - diff vs upstream/master = **B/C만 10파일 +832/-6**. group A 0.
  - **★ review가 확정해야 할 게이트(impl이 confirm 못함)**:
    - **G5 동등성**: upstream `SignalParamType` ⊇ 우리 18종(+faction) superset 주장 → review 부록기준 독립판정해야 폐기 확정. 갭없으면 C보강 불요.
    - npm test `other.literal.ts:11 "cannot infer list type"` 실패 = impl "upstream 선존버그, 순수 upstream worktree서 동일재현" 주장 → review 독립 재현.
    - signal e2e = `assert-signal-parameters.ts` L190 fixture(`src/resources/signals.ts`, upstream 미커밋) 게이트로 부분PASS(핵심 defineSignal→GS변환+typed evt.params 통과). review가 fixture 확보 시 300000/300001 핀 완주 권장.
  - **★ 02:4x review dispatch 완료**: spec-review-017.md REV(R3 B1분산 합산판정 + R4 npm-test/e2e fixture 독립재현 + R5 동등성게이트 + R6 마이그레이션델타) → tree-cli send TASK_ASSIGN (unread.20260607T0241) → **DEAD** → leader 활성화 SPAWN_REQUEST(Name: fork-sync~review) 송부 완료.
  - **현 상태: review TASK_RESULT 대기 중** (audit → `notes/integration/upstream-merge-audit.md`).
  - **다음**: review 수령 → fork-sync 사후확인(특히 R5 동등성 판정 채택여부) → verdict-017 작성(브랜치 integ-upstream / 태그 fork-backup-20260607 / NEW_BASE=9cb31c8 / G2처리표 / §4grep B1분산설명 / §5증거 / 폐기 review확인 / **마이그레이션델타표** / npm-test 선존버그) → leader TASK_RESULT 보고. master force·커밋은 leader 수용 시 별도(절대 내가 안 함).

---

## ▶ (이전) RESUME BOOKMARK (2026-06-07 01:0x, gen 교체 직전)
**나 = fork-sync (distributor), parent=leader. 역할: fork 유지보수 총괄, 구현(fork-sync~impl)·리뷰(fork-sync~review) 분할 조율.**
- **핵심 설계(leader)**: per-directive 신규노드 금지. 영속 `fork-sync~impl`/`fork-sync~review` 쌍을 **모든** fork-maintenance 작업에 재사용. 둘 다 보통 DEAD라 tree-cli send TASK_ASSIGN으로 인박스 seed 후 → leader에 `[SPAWN_REQUEST] Name: fork-sync~<impl|review>` SendMessage로 활성화 요청(직접 DM 금지).
- **완료된 작업(전부 PASS 종결, 보고완료)**:
  - #014 (베이스라인 번들 + integration-workflow): verdict-014.md. notes/integration/patches/ 20패치 + README + integration-workflow.md + review-audit.md.
  - #015 (gil_scaffold.ts 2버그 + import fix, 커밋 594a6e8): verdict-015.md.
  - #016 (#015 후 C2패치 refresh 236→251): verdict-016.md.
- **현재 상태**: 미결 작업 없음. **새 디렉티브/지시 대기 중.** 들어오면 위 재사용 패턴으로 impl→review 분할.
- **커밋 대기(leader 처리)**: notes/integration/{patches/C2-cli-gil_scaffold.patch, patches/README.md, review-audit.md, baseline-refresh-audit.md} — verdict-016.md "커밋 안내" 참조.
- **주의/함정**: ① SPAWN_REQUEST는 persistent type 아님 → tree-cli send 불가, SendMessage로 leader 직접. ② worktree apply --check batch 테스트 시 절대경로 쓸 것($PWD가 subshell cd 후 재평가되는 버그 주의). ③ 형제 fork-steward는 read-only(조율 안 함). ④ 분기점 71ca7a1, 권위델타=git diff 71ca7a1 HEAD(현 +1037/-26, HEAD=594a6e8).
- 상세 이력은 아래 섹션들 + workspace/verdict-01{4,5,6}.md.

---

## Task
**Directive #014** (standing) — fork-maintenance org 총괄. 4축: 우리변형유지 / 코드리뷰분리 / upstream변형최소화 / 통합지속.
- Directive: `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts\.claude\tree\gsts\directives\014-fork-sync-mission.md`
- **First work item** (지금 진행): upstream URL 없이 가능한 통합 베이스라인 & 워크플로 구축.
  - impl 노드: signal-args HIGH 패치셋 → 재적용 가능 패치 번들 + `notes/integration-workflow.md` 초안.
  - review 노드: 번들/문서 독립 검수 (변형 누락無 / 버그수정 무결성 / 충돌위험도 검증).
  - 검수 통과분만 verdict로 묶어 leader 보고.

## Task #015 (신규, 2026-06-06 23:48) — scaffold codegen 버그 2개 수정
- Directive: `.claude\tree\gsts\directives\015-scaffold-codegen-fix.md`
- 대상: `src/cli/gil_scaffold.ts` `buildDefaultValue()` 부근. (C카테고리 우리 고유 CLI — upstream 충돌 없음)
- 버그1: vec3 이중래핑 `vec3(vec3(0,0,0))`. 버그2: list/dict bare-comment `subParts: /* entity_list */,` → 컴파일 불가.
- **#014와 차이: src 수정함 + 실제 실행검증 필수** (회귀 .gil scaffold → tsc 컴파일 통과).
  - 회귀 .gil: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil` (AstroGear_05_Aegis id 1073741920: vec3+entity_list 둘 다 보유).
  - 임시작업 R: 램디스크 가능. repo 루트 더럽히지 말 것.
- 제약: 다른 버그수정(varint guard 등)/기존동작/upstream공유코드 미수정. impl≠review 분할.
- **시퀀싱 결정: #014 review 사이클 먼저 종결 후 #015 dispatch.** 이유: #015가 gil_scaffold.ts 수정 → #014 베이스라인 번들의 C2-cli-gil_scaffold.patch가 stale 됨. #014 스냅샷을 먼저 확정·보고한 뒤 파일 변경 → 스냅샷 일관성 유지. (leader가 순서 자율 허용.)
- **#014 verdict에 반드시 기재**: #015 적용 후 gil_scaffold.ts 베이스라인 번들 갱신 필요(C2 패치 refresh).
- #015 specs는 지금 작성(대기비용 0), dispatch는 #014 종결 후.
- **#015 산출물 작성 완료 (대기 중)**:
  - 계약: `workspace/contract-015-scaffold-fix.md` (D1~D5 검수기준, 회귀.gil 경로).
  - impl 스펙: `workspace/spec-impl-015.md` (버그 정확 라인: buildDefaultValue L56 vec3 재래핑, L70 default bare주석).
  - review 스펙: `workspace/spec-review-015.md` (D4 독립 재컴파일 필수).
  - 버그 코드 확인됨: gil_scaffold.ts L52~72 buildDefaultValue, L20~35 TYPE_TO_CONSTRUCTOR(entity_list→'list' L28).
- **#015 dispatch 트리거**: #014 verdict를 leader에 TASK_RESULT 보고 직후 → SPAWN_REQUEST(impl-015) → leader.

## Org 분할 (반드시 준수)
- 구현 노드 ≠ 리뷰 노드 (자가검수 금지, 관점 전환 검수).
- 트리 깊이 2단계까지. 자식은 SPAWN_REQUEST로 leader에 요청.
- 산출물은 `notes/`에만 (src 미수정 — 준비/문서화 단계).
- 버그수정(B1 varint guard / B2 any 타입가드 / B5 parseValue entity fallback) 절대 무심코 롤백 금지.

## 분할 계획
- Child `impl` (opus): patch bundle + integration-workflow.md → `notes/integration/`.
  - spec: `workspace/spec-impl.md`
- Child `review` (opus): impl 산출물 독립 검수 → audit 문서.
  - spec: `workspace/spec-review.md` (impl 완료 후 송부)
- 공유 계약: `workspace/contract-deliverables.md` (산출물 경로/형식/검수기준).

## Refs
- `notes/fork-changes-inventory.md` — 포크 변경 전체 인벤토리 (충돌 위험도). 필독 완료.
- 분기점: upstream `71ca7a1` 위 선형. `git diff 71ca7a1 HEAD` = 권위 델타 (84 files, +10604/-27).
- HIGH 핫스팟 7: core.ts, nodes.ts(sendSignal+parseValue), mappings.ts, index.ts, pins.ts, signal_nodes.ts.

## 진행 상태
- [x] 인벤토리 정독, 권위 델타 검증 (git diff 71ca7a1 HEAD = 20 src, +1022/-26, 인벤토리와 일치).
- [x] 계약 작성: `workspace/contract-deliverables.md` (경로/형식/검수기준 C1~C5).
- [x] impl 스펙: `workspace/spec-impl.md`.
- [x] review 스펙: `workspace/spec-review.md` (impl 완료 후 송부 — impl 산출물에 의존).
- [x] SPAWN_REQUEST(impl) → leader 송부 (2026-06-06).
- [x] **impl TASK_RESULT 수령** (23:45). 디스크 검증 OK: notes/integration/patches/ 20패치+README, notes/integration-workflow.md 존재, src clean. C4 impl 실증(temp worktree git apply --check 개별+일괄). C1/C2/C3/C5 self-claim — review 판정 대기.
  - impl result: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result.md`
- [x] SPAWN_REQUEST(review) → leader 송부 (2026-06-06).
- [x] **review TASK_RESULT 수령** (23:50). verdict PASS, C1~C5 전부 PASS, 치명 0, 경미 R1/R2(비차단). audit: `notes/integration/review-audit.md`.
- [x] **fork-sync 독립 재확인**: 20패치 apply --check vs HEAD = FAIL 20/20(정상, 분기점에서만 적용), B2/B5/B1 가드 grep 일치 → review PASS 채택.
- [x] **통합 verdict 작성**: `workspace/verdict-014.md` (R1/R2 권고 + #015 C2패치 stale 주의 포함).
- [x] **#014 → leader TASK_RESULT 보고 완료** (verdict-014.md). **#014 First item 종결.**

## #014 종결 ✅ — 이후 #015로 전환

## ★ 설계 정정 (leader 지시)
- **per-directive 신규 노드 만들지 말 것.** 영속 `fork-sync~impl`/`fork-sync~review` 쌍을 **모든** fork-maintenance 디렉티브에 재사용. 노드=영속 도메인 소유자, 일회성 작업은 재사용.
- #015도 신규 impl-015 SPAWN 취소 → 기존 fork-sync~impl/~review에 tree-cli send TASK_ASSIGN.
- 스펙의 "impl-015/review-015"는 역할 라벨일 뿐. 실행노드=fork-sync~impl/~review. result는 각 노드 자기 workspace/. (specs/contract 경로 정정 완료.)

## #015 진행 상태
- [x] 계약+impl/review 스펙 작성·경로정정 (contract-015-scaffold-fix.md, spec-impl-015.md, spec-review-015.md).
- [x] (취소) impl-015 신규노드 SPAWN → leader 재사용 지시로 정정.
- [x] **#015 impl 스펙 → fork-sync~impl 인박스 TASK_ASSIGN seed** (tree-cli send, unread.20260606T2357). 노드 DEAD → leader에 활성화 SPAWN_REQUEST(Name: fork-sync~impl) 송부 완료. impl TASK_RESULT 대기 중.
- [x] **impl TASK_RESULT 수령** (00:12). result: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-015.md`. 2버그 수정 + 회귀.gil scaffold→tsc EXIT=0 실증(baseline TS1109 실패). D1~D5 self-claim.
- [x] **fork-sync D5 독립확인**: `git status --short` = tracked 변경 `M src/cli/gil_scaffold.ts` 단 1개(31+/16-). 나머지 untracked는 .claude/+notes/만(코드 stray 0, 루트 미오염). D5 holds.
- [x] **★ impl이 3번째 수정 escalate**: import emit 재작성(`{g,...} from 'genshin-ts'` → `{g} from 'genshin-ts/runtime/core'`, 값생성자=ambient global이라 import 불필요 주장). 명시 2버그 외지만 D4(tsc통과) 달성 필수라는 판단. → 내 판단: 동일파일+디렉티브 Goal("유효 컴파일가능 TS") 직결 → **in-goal로 잠정수용**, review가 D6으로 독립확정.
- [x] **review 스펙에 D6 추가** (3번째수정 (a)사실검증 (b)불가피성 (c)부작용). spec-review-015.md.
- [x] **#015 review 스펙 → fork-sync~review TASK_ASSIGN seed** (unread.20260607T0013). 노드 DEAD → leader 활성화 SPAWN_REQUEST(Name: fork-sync~review) 송부 완료. review TASK_RESULT 대기 중.
- [x] **review TASK_RESULT 수령** (00:36). verdict PASS, D1~D6 전부 PASS, 치명 0, INFO N1/N2. audit: `notes/integration/scaffold-fix-audit.md`. D4 인과 실증(fixed EXIT=0 vs pre-fix TS1109 EXIT=2), D6 (a)(b)(c) 전부 독립확인 → 3번째수정 in-goal 확정수용.
- [x] **fork-sync 독립 재확인**: g=core.ts:910 export, scaffold가 corrected import emit(collectImports 제거), 신규 코드경로 소스 존재 확인 → review PASS 채택.
- [x] **통합 verdict-015 작성**: `workspace/verdict-015.md`.
- [x] **#015 → leader TASK_RESULT 보고 완료. #015 종결.**

## #015 종결 ✅ (PASS)

## ★ Follow-up 작업 (#016 refresh) — #015 커밋됨, 진행 중
- **트리거 발동**: leader가 #015 커밋 보고(HEAD=`594a6e8`). C2 패치 이제 진짜 stale.
- fork-sync 확인: `git diff --stat 71ca7a1 HEAD -- src/cli/gil_scaffold.ts` = **251 insertions**(구 236→신 251; 분기점=신규파일이라 del 0, #015 +31/-16 누적). 번들 20패치.
- 분할: impl이 C2 refresh + 번들 일관성, review가 독립 검증(베이스라인 무결성 = 머지 기준자료라 신중).
- [x] impl 스펙 `workspace/spec-impl-016-refresh.md`, review 스펙 `workspace/spec-review-016-refresh.md` 작성.
- [x] impl 스펙 → fork-sync~impl TASK_ASSIGN seed → leader 활성화 요청.
- [x] **impl TASK_RESULT 수령** (00:56). result-016.md. C2 251줄 재생성(cmp 일치), 19패치 MATCH/C2 갱신, apply --check 20/20+일괄 OK, 트리무오염. README delta +1022→+1037, #015 커밋 추가. E1~E4 self-claim.
- [x] **fork-sync 독립 재확인**: (E1) C2=`git diff 71ca7a1 HEAD` cmp MATCH. (E2) 20패치 전수 cmp = 20 MATCH(C2포함). (E3) 임시worktree apply --check 개별 20/20 + 일괄 OK, src 무변경·worktree 메인만. → impl claim 전부 확인.
  - ⚠️ 내 첫 batch테스트 FAIL은 **내 스크립트 버그**(`$PWD`가 subshell cd 후 worktree로 재평가). 절대경로로 재실행하니 OK. 패치 문제 아님 — 다음 세대 재오인 금지.
- [x] **impl 플래그 잔여(스코프 밖, 합당)**: review-audit.md L5 stale `+1022`(review 소유), inventory/changelog의 236(역사적 기록) — #016 번들 스코프 밖, impl 미수정 정당.
- [x] **review 스펙 → fork-sync~review TASK_ASSIGN seed** (unread.20260607T0058). 노드 DEAD → leader 활성화 요청 송부.
- [x] **review TASK_RESULT 수령** (01:04). verdict PASS, E1~E4 전부 PASS. audit: `notes/integration/baseline-refresh-audit.md`. 독립 cmp 20/20 IDENTICAL + worktree apply OK + C2 #015마커 확인. F1(review-audit L5) 자기갱신→+1037, F2 역사적기록 스코프밖.
- [x] **fork-sync 사후 확인**: C2 #015마커 7건, README +1037 & #015커밋 & refresh이력, review-audit L5 +1037 갱신 — 전부 확인.
- [x] **통합 verdict-016 작성**: `workspace/verdict-016.md`.
- [x] **#016 → leader TASK_RESULT 보고 완료. #016 종결.**

## #016 종결 ✅ (PASS) — 베이스라인 HEAD=594a6e8 권위델타와 완전정합

## 커밋 대상 (leader 세션마무리 커밋용, verdict-016에 기재)
- notes/integration/patches/C2-cli-gil_scaffold.patch (refresh 251)
- notes/integration/patches/README.md (+1037, #015커밋, 이력)
- notes/integration/review-audit.md (L5 +1037)
- notes/integration/baseline-refresh-audit.md (신규)

## 대기 상태
- 새 디렉티브/지시 대기. 영속 fork-sync~impl/~review 재사용(신규노드 금지).
- #014/#015/#016 전부 PASS 종결. 베이스라인 번들 = 현 HEAD 권위델타 정합.

## Notes
- 2026-06-06: impl/review 분할. impl 먼저 spawn (review는 impl 산출물 의존). Org 제약: 구현≠리뷰 노드.
- 주의: SPAWN_REQUEST는 tree-cli send 불가(persistent type 아님) → SendMessage로 leader에 직접.
- 사이드노트: 형제 `fork-steward`가 거의 동일 역할 — 형제는 read-only, 조율 안 함. leader가 #014를 나에게 배정 → 내 서브트리로 진행.
