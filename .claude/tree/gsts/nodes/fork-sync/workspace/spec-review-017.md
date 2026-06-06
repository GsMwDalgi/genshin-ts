# Task Spec — fork-sync~review — Directive #017 Upstream Merge (독립 검수)

> 실행 노드 = `fork-sync~review` (영속). audit → `notes/integration/upstream-merge-audit.md`.
> 계약: `.claude/tree/gsts/nodes/fork-sync/workspace/contract-017-upstream-merge.md` (먼저 정독).
> impl 결과: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-017.md` (claim — **신뢰 금지, 독립 재실행**).
> 권위 절차: `notes/integration-workflow.md` §4/§5 + 부록.

## ★ 방향 = B 확정 (leader+user). 너의 핵심 임무 = 폐기 정당성 게이트
- impl이 group A(A1~A13 signal-args)를 **폐기**하고 upstream `defineSignal`/typed `evt.params` 채택함. **이 폐기를 확정하려면 너의 독립 동등성 확인이 게이트**(Org 제약: 폐기 결정은 review 독립검증 후). 아래 R5가 이번 검수의 중심.
- 동등성 미충족 갭 발견 시 → 즉시 FAIL이 아니라 "갭 X는 C-하이브리드 보강 필요"로 명기(우리 기능 손실 방지 — leader가 갭만 보강 지시). 갭 없으면 폐기 확정 PASS.

## 역할
impl이 실행한 upstream 머지(방향 B)를 **독립 관점**에서 검수. impl 로그를 그대로 믿지 말고, grep·검증을 직접 재실행하고 작업 브랜치를 직접 검사한다. 자가검수 아님(impl≠review).

## 사전 — G0 게이트 확인
- impl result-017.md의 G0(NEW_BASE 판정) 먼저 확인.
- **impl이 "NEW_BASE==71ca7a1, 머지할 것 없음"으로 중단했으면**: NEW_BASE 해시가 정말 71ca7a1과 동일한지 `git fetch upstream` + `git rev-parse upstream/<tip>`로 **독립 확인**만 하고 audit에 PASS/근거 기록. R1~R5 N/A.
- 머지를 진행했으면 → R1~R5.

## 검수 항목 (각 PASS/FAIL/INFO + 독립 근거)

### R1 — 가역성 (G1)
- 백업 태그가 옛 HEAD(`8629dc9`)를 가리키는지: `git rev-parse fork-backup-<date>` == `8629dc9`.
- `git rev-parse master` == `8629dc9` (master 불변 — force 안 됨).
- 작업 브랜치(`integ-upstream` 등) 존재.

### R2 — 방향 B 적용 정합성 (G2) — A 폐기 + upstream 채택 + B/C 재이식
작업 브랜치 체크아웃(temp worktree 권장, 루트 오염 금지) 후 확인. **주의: 방향 B라 우리 group A는 폐기됨 — A 코드가 남아있으면 오히려 이상**(중복).
- **A 폐기 확인**(우리 명명이 코드에 없어야 정상): `git grep -nE "signalArgs|SignalArgsToPayload|SignalArgDef|SIGNAL_ARG_TYPE_MAP" -- src/` → 우리 A 잔재 0 기대(있으면 폐기 불완전 INFO/FAIL).
- **upstream signal 채택 확인**: core.ts `defineSignal`/`onSignal<S extends SignalDefinition>` 존재, mappings.ts `send_signal:300000` 등 upstream 인코딩 존재.
- **B/C 재이식 확인**(아래 R3에서 grep, 여기선 구조): nodes.ts에 B5(entityLiteral), signal_nodes.ts에 B1 가드가 upstream 새 구조 위에 얹혀 있는지. C(gil_inspect/gil_scaffold/reader.ts 신규파일 + gsts.ts 서브커맨드 등록) 존재.
- impl의 G2 처리표와 실제 코드 대조. A 잔재·B/C 누락 없는지.

### R3 — B1/B2/B5 무결성 (G3/§4, 절대 롤백금지) — 독립 grep 재실행
- **★ B1 위치 분산 주의(upstream 구조변경)**: upstream v0.1.10이 `readFieldBytes`/`readFieldMessages`를 signal_nodes.ts → **binary.ts로 이동**함. 따라서 §4의 "signal_nodes.ts 8건" 기대는 더 이상 단일파일로 안 맞음. **두 파일 합산**으로 확인:
  - `grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/signal_nodes.ts` → 2건 기대(parseNodeGraphId: offset>=0 루프가드 + newOffset<0).
  - `grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/binary.ts` → 7건 기대(readFieldBytes+readFieldMessages 각 offset>=0/len<0/dataEnd<0 = 6 + upstream parseMessage 기존 1).
  - **판정 기준 = 가드 의미가 3함수(readFieldBytes/readFieldMessages/parseNodeGraphId) 전부에 보존**되는지(파일 위치 무관). 합산 9건 + 함수별 가드 존재 = PASS. signal_nodes.ts 단독 2건만 보고 "롤백됐다"고 오판 금지(fork-sync 독립확인에서 이미 위치분산 검증됨).
- B2: `grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts` → 1건(isEntityLikeType 첫줄).
- B5: `grep -n "new entityLiteral" src/definitions/nodes.ts` → 1건 이상(parseValue case 'entity').
- impl 카운트(result G3 표: B1 signal_nodes 2 / binary 7 / 합 9, B2 1, B5 1)와 일치하는지.

### R4 — §5 검증 독립 재실행 (G4)
- 작업 브랜치에서 직접: `npm install` → `npx tsc --noEmit`(또는 정의 스크립트) **EXIT 코드 직접 확인(0)** → `npm run build`(EXIT 0).
- injection smoke 최소 1종 독립 확인: `gsts inspect`/`gsts scaffold` on 샘플 .gil `1073741976.gil`이 동작하는지(scaffold 산출 tsc 통과; 출력은 temp/R: — 루트 오염 금지).
- **★ npm test EXIT=1 검증(중요)**: impl이 `npm test`가 `tests/.../other.literal.ts:11 "cannot infer list type"`에서 EXIT1나는데 **"upstream v0.1.10 자체 선존버그, 우리 머지와 무관"**이라 주장(순수 upstream worktree서 동일 재현). **독립 재현 필수**: `git worktree add <temp> upstream/master` → npm install → npm test → 동일 EXIT1·동일 메시지인지 확인. 동일하면 우리무관 PASS(INFO 기록). 우리 트리에서만 나면 FAIL(머지가 깬 것).
- **★ signal-args e2e 부분PASS 검증**: impl이 `assert-signal-parameters.ts`를 돌려 핵심(defineSignal→GS변환+typed `evt.params`+sendSignal 어서션)은 통과, L190에서 로컬 fixture `src/resources/signals.ts`(upstream 미커밋) 부재로 중단이라 보고. 독립 재실행해 (a)핵심 단계 통과 확인 (b)L190 게이트가 우리 변경과 무관(fixture 부재)임을 확인. 가능하면 fixture를 우리 .gil(`gsts inject`/`extractSignalsFromGil`)로 생성해 300000/300001 핀 깊이검사까지 완주 권장(불가 시 부분PASS+사유).
- impl 증거(result G4 표)와 대조. 불일치 시 FAIL.
- ⚠️ batch/worktree 테스트 시 절대경로 사용($PWD가 subshell cd 후 재평가되는 함정 — memo 기록된 과거 오인 주의).

### R5 — ★ group A 폐기 동등성 게이트 (G5/부록) — 이번 검수의 중심
impl이 group A(A1~A13)를 폐기·upstream `defineSignal` 채택. 폐기 확정 = 너의 독립 동등성 확인(Org 제약). 부록 기준:
- (1) **18종 타입 커버리지**: 우리 옛 `SIGNAL_ARG_TYPE_MAP` 18종(entity/int/str/vec3/*_list 등)을 upstream signal 타입/핀인코딩이 모두 표현 가능한지 — upstream 정의 직접 검사 + 18종 대조표.
- (2) **`_list` 자동래핑**: 우리 옛 A8 assemblyList 자동래핑 동작을 upstream이 등가 제공하는지.
- 판정:
  - 전부 동등 → **폐기 확정 PASS**.
  - 갭(일부 타입/_list 미지원) → FAIL 아님: "갭 X → C-하이브리드 보강 필요"로 명기(기능손실 방지, leader가 갭만 보강 지시).
  - 동등성 **미검증/검증불가**인데 폐기 → **치명 FAIL**(미검증 폐기 = Org 제약 위반).
- B1/B2/B5는 폐기 아님(보존) — R3에서 확인. upstream이 동등 도입했으면 INFO로만 기록.

### R6 — 마이그레이션 델타 정확성 (leader 요구)
- impl result의 "마이그레이션 델타 표"(우리 옛 API → upstream 등가) 정확성 독립 확인:
  - `sendSignal(...signalArgs)` / `onSignal<Args>` / `SIGNAL_ARG_TYPE_MAP` → upstream 등가 매핑이 맞는지.
  - 누락/부정확 시 INFO/FAIL 명기(downstream gsts-sandbox/팀git/mwe 마이그레이션 자료 — 정확성 중요).

## audit 작성 (`notes/integration/upstream-merge-audit.md`)
- 각 R 항목 PASS/FAIL/INFO + 독립 근거(grep 출력/EXIT 코드/git rev-parse).
- 치명(blocking) vs 경미(non-blocking) 구분.
- 파일 끝 verdict: PASS / FAIL + 사유.

## escalate
- impl이 §4 가드를 떨어뜨렸거나(B1/B2/B5 누락), master를 force했거나, 동등성 미확인 폐기 등 치명 발견 시 → audit에 FAIL 명기. fork-sync(parent)가 rework 판단.
