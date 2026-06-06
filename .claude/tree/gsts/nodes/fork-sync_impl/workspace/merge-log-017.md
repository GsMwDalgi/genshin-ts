# Merge Log — Task #017 Upstream Merge (impl)

> 실행 노드: fork-sync~impl. 진행 로그 + STEP별 증거. 본 머지는 **STEP1 도중 escalate**로 일시중단(rebase abort, 가역).
> 결정 대기 항목은 `workspace/escalation-017.md`. self-eval(G0~G5)은 result-017.md.

## STEP 0 — 사전 + NEW_BASE (G0/G1)

| 항목 | 값/결과 |
|------|---------|
| 워킹트리 code-clean | OK — tracked src 변경 0. `.claude/` tree-state만 변경(무시 대상). |
| 백업 태그 | `fork-backup-20260607` @ `8629dc9` (현 HEAD). 생성 확인. |
| upstream remote | `git remote add upstream https://github.com/josStorer/genshin-ts` OK |
| `git fetch upstream` | OK — `upstream/master` + v0.1.1~v0.1.10 태그 수신 |
| upstream 기본브랜치 | `master` (`git remote show upstream` → HEAD branch: master) |
| **NEW_BASE** | `9cb31c8` = "release v0.1.10" |
| 우리 분기점 | `71ca7a1` |
| 게이트 판정 | NEW_BASE(`9cb31c8`) **!=** 71ca7a1 → upstream 전진함 → 머지 진행 (게이트 통과, 중단 아님) |

추가 위상 분석:
- `git merge-base --is-ancestor 71ca7a1 upstream/master` → **YES**. 71ca7a1은 upstream tip의 진짜 조상. merge-base(71ca7a1, upstream/master) == 71ca7a1.
- `git rev-list --count 71ca7a1..upstream/master` = **43** (upstream이 43커밋 전진).
- 우리 커밋 레인지 `git log --oneline 71ca7a1..HEAD` = **9 커밋**:
  `eda452b, 66f4803, c16dfa7, d2320e8, 9415773, a216ac9, 5da1cb4, 594a6e8, 8629dc9`
  (66f4803 = 대형 feature 마이그레이션; 나머지는 chore/docs/.claude 상태 + #015 scaffold fix)
- → 선형 fork-on-top → `git rebase --onto upstream/master 71ca7a1 integ-upstream`가 정석.

## STEP 1 — 작업 브랜치 + rebase 시도 (G1)

| 항목 | 결과 |
|------|------|
| `git checkout -b integ-upstream` | OK (master에서 분기) |
| master 불변 | `git rev-parse master` = `8629dc9` (그대로) |
| `.claude/` tree-state | rebase 전 `git stash push -u -- .claude/`로 격리(코드와 무관), abort 후 `git stash pop`로 복원 |
| rebase 명령 | `git rebase --onto upstream/master 71ca7a1 integ-upstream` |
| 결과 | 커밋 1/9(66f4803)에서 **CONFLICT** → 7파일 충돌 |

### rebase 충돌 7파일 (커밋 66f4803 적용 중)
1. `src/runtime/core.ts` — UU
2. `src/definitions/nodes.ts` — UU
3. `src/compiler/ir_to_gia_transform/index.ts` — UU
4. `src/injector/signal_nodes.ts` — UU
5. `src/cli/gsts.ts` — UU (HIGH-7 밖, 서브커맨드 등록 충돌)
6. `src/runtime/server_globals.d.ts` — UU (HIGH-7 밖)
7. `src/runtime/server_globals.ts` — UU (HIGH-7 밖)

자동머지 성공(충돌 없음)한 우리 HIGH-7 파일: `runtime/value.ts`(A01 — SignalArgDef/SignalArgsToPayload export 보존 확인), `mappings.ts`(A10), `pins.ts`(A12).

### core.ts 부분해소 (참고 — abort로 폐기됨)
- core.ts 충돌은 value.js import 블록 2곳뿐(양측이 서로 다른 named import 추가). union 병합으로 해소 시도(generic/float/guid/int/vec3 + RuntimeParameter/ReturnValueTypeMap + SignalArgDef/SignalArgsToPayload 모두 포함). 18개 심볼 전부 value.ts에 존재 확인.
- **그러나** core.ts 본문(onSignal)은 자동머지가 **upstream판 onSignal을 채택하고 우리판을 드롭** → 이게 escalate 발견의 발단(아래). 따라서 부분해소는 무의미 → rebase abort.

## 중대 발견 → ESCALATE (rebase abort)

자동머지 결과 우리 `onSignal<Args extends readonly SignalArgDef[]>`이 사라지고 upstream의 `onSignal<S extends SignalDefinition>`이 자리잡음. 조사 결과 **upstream v0.1.10이 signal-args 전체를 독립 도입**했음을 확인. 상세·증거·결정요청은 `workspace/escalation-017.md`.

reversibility 조치:
- `git rebase --abort` 실행 → integ-upstream이 master(8629dc9)와 동일 상태로 복귀.
- `git stash pop`으로 .claude tree-state 복원.
- 검증: `git rev-parse master` = `8629dc9`, `git rev-parse fork-backup-20260607` = `8629dc9`. 둘 다 무손상.

## 현 리포지토리 상태 (escalate 시점)
- 브랜치: `integ-upstream` 존재(HEAD == master == 8629dc9, rebase abort로 빈 상태).
- `master` = `8629dc9` (force 안 함).
- 백업 태그 `fork-backup-20260607` = `8629dc9`.
- `upstream` remote 등록됨(`upstream/master` = `9cb31c8`).
- src 워킹트리 무오염(충돌 마커 잔존 없음 — abort로 제거). `.claude/` tree-state만 변경.
