# 통합 Verdict — Directive #015 (scaffold codegen 2 bug fix)

> 작성: fork-sync (distributor) / 2026-06-07.
> impl 노드(fork-sync~impl) 구현 + review 노드(fork-sync~review) 독립 검수를 종합.
> Org 제약 준수: 구현·리뷰 분리, 관점 전환 검수, review의 독립 재컴파일로 self-claim 검증.

## 종합 판정: **통과 (PASS) — 조건 없음**

`src/cli/gil_scaffold.ts`의 scaffold 코드젠 결함이 해결되어, 변수 보유 .gil scaffold 출력이 유효·컴파일 가능한 TS가 됨(tsc EXIT=0 실증, pre-fix는 TS1109 실패로 인과 확인). 명시 2버그 + 불가피한 3번째 결함(import emit)까지 in-goal로 처리. 치명/중대 이슈 0, INFO 2건(비차단).

## 변경 (단일 파일, src)
- `src/cli/gil_scaffold.ts` (+31/-16, 워킹트리 — 미커밋). 그 외 src 미수정.
  - 버그1(D1): `normalizeVec3()` 신규 + vec3 initialValue 분기 → `vec3([x,y,z])` 단일·유효(구: `vec3(vec3(0,0,0))` 이중래핑 & arity오류).
  - 버그2(D2): switch에 `dict→dict(0)`, `*_list→list('<elem>',[])` 추가 → 무효 bare-comment 제거.
  - 3번째(D6): import emit 재작성 `import {g,...} from 'genshin-ts'` → `import { g } from 'genshin-ts/runtime/core'` + `collectImports` 제거. (값 생성자는 ambient global이라 import 불필요.)

## 자식 결과 종합 (각 self-eval을 fork-sync가 검증)
- **impl (fork-sync~impl)** — claim: D1~D5 충족(D4 실증), 3번째 수정 escalate. result: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-015.md`.
- **review (fork-sync~review)** — verdict: PASS, D1~D6 전부 PASS, 치명 0, INFO 2(N1/N2). audit: `notes/integration/scaffold-fix-audit.md`.
  - 독립성: impl 주장 비신뢰, scaffold+tsc 직접 재실행. **D4 인과 실증**(fixed tsc EXIT=0 vs pre-fix baseline TS1109 EXIT=2, 동일 .gil). D6은 (a)사실(g=core.d.ts:233 export, 값생성자=server_globals.global.d.ts:28 ambient, root genshin-ts에 값export 0)·(b)불가피(없으면 변수보유 scaffold 무조건 TS2305)·(c)무부작용 전부 확인.

## fork-sync 독립 재확인 (review claim을 신뢰만 하지 않고 검증)
- D5 범위: `git status --short` tracked 변경 = `M src/cli/gil_scaffold.ts` 단 1건. stray 코드 0, repo 루트 미오염(나머지 untracked는 .claude/+notes/).
- D6 사실 핵심: `src/runtime/core.ts:910 export const g`, gil_scaffold.ts가 `import { g } from 'genshin-ts/runtime/core'` emit(collectImports 제거 확인), 신규 코드경로(normalizeVec3/`vec3([..])`/`dict(0)`/`list('<elem>',[])`) 소스에 존재 확인.
- → 양 자식 합치 + 독립 재현 + 내 spot-check 일치 → PASS 채택. D6 3번째 수정 **확정 in-goal 수용**(동일파일·디렉티브 Goal 직결·불가피·무부작용).

## 디렉티브 판정 (Goal/Constraints/Verification)
- Goal("어떤 변수 타입이든 유효·컴파일 가능 TS"): **달성** — vec3/`*_list`/dict/스칼라 전 분기 tsc EXIT=0.
- Constraints: 버그수정 신중·구현≠리뷰 분리 ✔ / 다른 버그수정·기존동작 미접촉 ✔ / 범위 gil_scaffold.ts 한정·upstream 공유코드 미수정 ✔ / repo 루트 미오염(R: 램디스크 사용) ✔.
- Verification(실제 실행): 회귀 .gil(AstroGear_05_Aegis id 1073741920) scaffold→tsc EXIT=0, 타입별 샘플 검증 ✔.

## 후속 / 주의 (INFO — 차단 아님)
- **N1 — #014 베이스라인 C2 패치 refresh (leader 지시 챙김)**: `notes/integration/patches/C2-cli-gil_scaffold.patch`는 `71ca7a1..HEAD` 스냅샷. **현재는 아직 stale 아님** — #015 변경이 **미커밋(워킹트리)**이라 `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`는 여전히 구 236줄 델타.
  - 조치 타이밍: **#015가 커밋된 후** C2 패치를 `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`로 재생성(refresh). 지금 미리 갱신 금지(미커밋 상태 반영 안 됨). → 커밋 시점 알림 필요 시 fork-sync가 처리.
- **N2 — normalizeVec3 견고성**: reader 출력형식(`vec3(x,y,z)`/`vec3([..])`) 의존. 현재 안전, 향후 reader 형식 변동 시 재확인. 조치 불요.

## self-eval (directive #015 기준)
criteria met: 버그1 해결 YES · 버그2 해결 YES · 유효 컴파일가능 TS(Goal) YES · 실제 tsc 실증(인과) YES · 구현≠리뷰 독립검수 YES · 범위 한정·루트 미오염 YES. 3번째 수정 in-goal 확정. 치명 0, INFO 2(비차단). **PASS.**
