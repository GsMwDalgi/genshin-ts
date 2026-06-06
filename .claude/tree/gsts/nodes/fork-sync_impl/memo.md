# Memo

> RESUME-BOOKMARK (2026-06-07): **Task #017 방향B 실행 완료 — result 보고 단계.** [이전 escalate→leader+user 방향 B 확정 수신]. 실행: integ-upstream를 upstream/master(9cb31c8)로 reset → group A(A1~A13) 폐기(재적용 안함, upstream defineSignal이 대체) → B1/B2/B5 + C만 upstream 새 구조 위 재이식. **현 상태: integ-upstream HEAD=9cb31c8 + 워킹트리에 B/C변경 uncommitted**(커밋 안함 — leader 수용 후 단계). master=8629dc9 불변, 백업태그 fork-backup-20260607=8629dc9.
>
> 실행결과: src diff vs upstream = B/C만(10파일 ~832+/6-). **B1 재이식 구조변경 주의** — upstream이 readFieldBytes/readFieldMessages를 signal_nodes.ts→binary.ts로 이동 → B1 가드를 binary.ts(7건)+signal_nodes.ts parseNodeGraphId(2건)=총9에 분산. §4 grep on signal_nodes.ts만 보면 2건뿐(스펙 8건 기대와 다름) → review에 분산 명기 필수. B2 ts_type_utils.ts 1건; B5 nodes.ts entityLiteral 1건+import. C=gil_inspect/gil_scaffold/reader 신규 + gsts.ts inspect/scaffold 등록 + locale 10키×2(en/zh).
> 검증: npm install EXIT0(15pkg changed), npx tsc --noEmit EXIT0, npm run build EXIT0, scaffold tsc EXIT0(#015 픽스 vec3([])/list()/import 보존), inspect 회귀 184그래프 OK, scaffold 1073741920 OK. signal-args e2e: assert-signal-parameters.ts defineSignal→GS typed evt.params 단계 PASS(L174-188), L190서 로컬 fixture src/resources/signals.ts(upstream 미커밋) 부재로 중단=우리무관. **npm test EXIT1=other.literal.ts:11 "cannot infer list type" → PURE upstream worktree도 동일 실패 확인=upstream v0.1.10 자체 버그, 우리무관.** G5 동등성: upstream SignalParamType ⊇ 우리 18종 전부 + faction/faction_list 추가 = superset, 갭없음(폐기확정은 review 독립판정).
> 다음: result-017.md(마이그레이션델타표 필수 + G0~G5) → TASK_RESULT 보고. 스크래치/temp worktree 정리완료, 워킹트리 clean(B/C+package-lock+.vs editor cache).

## Task #017 (upstream merge 실행, 방향 B) — 실행완료·보고 2026-06-07
- spec REV2: `.../spec-impl-017.md`(방향 B), contract: `.../contract-017-upstream-merge.md`, directive Leader Decision §.
- 방향 B = group A 폐기 + upstream defineSignal 채택. B1/B2/B5+C 보존. 폐기확정은 review 동등성 게이트.
- result: `workspace/result-017.md`(마이그레이션델타표 + G0~G5 self-claim). merge-log: `workspace/merge-log-017.md`. (escalation-017.md = 이전 발견 기록.)
- 접근: integ-upstream를 upstream/master로 reset(rebase 아님 — A충돌 전수 회피) → B/C만 수동 재이식. A는 한 줄도 재적용 안함.
- **review에 넘길 핵심**: (1) B1 가드가 upstream 구조변경으로 binary.ts+signal_nodes.ts에 분산(signal_nodes 단독 grep 2건) (2) G5 동등성 superset 관찰(갭없음)이나 독립확인 필요 (3) npm test의 other.literal.ts 실패는 upstream 선존버그(worktree로 입증).
- 자가검수 금지: review가 G3/G4 독립재실행 + G2정합성 + G5폐기 독립확인.
- 검증증거 EXIT: install 0 / tsc --noEmit 0 / build 0 / scaffold-tsc 0 / npm test 1(upstream선존).

## Task #016 (C2 패치 refresh + 번들 일관성) — 완료 2026-06-07
- 배경: #015 커밋(594a6e8)으로 gil_scaffold.ts 누적 델타 236→251줄 → C2 패치 stale.
- 한 일: ① `notes/integration/patches/C2-cli-gil_scaffold.patch`를 `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`(251 insertions, new-file diff)로 재생성·덮어쓰기 — 동일 형식(new file mode 100644, blob 5a75f58). ② 20패치 전수 바이트대조: C2만 DIFFER, 나머지 19 MATCH(권위 델타와 일치). ③ 임시 worktree(71ca7a1)에서 20패치 개별+일괄 `git apply --check` 전부 OK → remove+prune. ④ README 갱신: line4 델타 +1022→+1037, 커밋목록에 594a6e8(#015) 추가, refresh 이력 1줄 추가.
- 검증 최종: refreshed C2 == `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts` 바이트 일치(cmp OK).
- 범위/오염: notes/integration/ 만 수정(C2 patch + README). src 미수정. R: 임시물 삭제, worktree prune, 메인트리 무오염.
- **플래그(review로)**: `notes/integration/review-audit.md` line5에 stale `+1022/-26` 잔존 — 그건 review 노드 산출물이라 내가 안 고침. inventory/changelog의 +236은 "원본 최초 추가 크기"로 역사적 기록(스코프 밖).
- result: `workspace/result-016.md`. 자가검수 금지(review 판정).
- Refs: spec `.claude/tree/gsts/nodes/fork-sync/workspace/spec-impl-016-refresh.md`, contract `.../contract-deliverables.md`, bundle README `notes/integration/patches/README.md`.

## Task #015 (scaffold codegen 2 bug fix) — 완료 2026-06-07
- 대상: `src/cli/gil_scaffold.ts` 한정 수정. result: `workspace/result-015.md`.
- 버그1 vec3 이중래핑: reader가 `vec3(x,y,z)`(3-arg 완성형) 반환 → buildDefaultValue가 재래핑 `vec3(vec3(...))`. 게다가 그 3-arg 형식 자체가 ambient global `vec3(value:Vec3Value)`(1-arg array)와 arity 불일치라 컴파일도 실패. 해결: `normalizeVec3()` 추가 → `vec3([x,y,z])` 단일·유효형. fallback `case 'vec3'`도 `vec3([0,0,0])`.
- 버그2 list/dict bare-comment: `*_list`→`list('<elem>',[])`, `dict`→`dict(0)`(유효 ReadonlyDict<never,never>). switch default에 `_list` 분기 + `case 'dict'` 추가.
- 추가 발견·수정(D4 필수): 기존 import `import {g,bool,...} from 'genshin-ts'` 전부 무효 — g/bool/vec3/list/dict는 genshin-ts root named export 아님(런타임·타입 모두 부재). g는 `genshin-ts/runtime/core` named export, 나머지는 ambient global(gsts types via server_globals.d.ts). → import를 `import { g } from 'genshin-ts/runtime/core'` 한 줄로 교체, collectImports 제거. 이거 없으면 어떤 scaffold도 tsc 불가 → 디렉티브 Goal/D4 충족 위해 in-scope(같은 파일) 수정.
- 검증(실증): 회귀 .gil(1073741976.gil, AstroGear_05_Aegis id=1073741920) CLI scaffold → tsc EXIT=0(스캐폴드 파일 0 에러). 전 타입분기(scalar/iv-scalar/*_list 7종/dict/vec3) 합성 샘플도 tsc 0 에러. gil_scaffold.ts 자체 src tsc clean.
- 범위/오염: src 변경=gil_scaffold.ts 1개뿐. R: 임시물 삭제, in-repo는 out/(gitignore) — 루트 미오염.
- 자가검수 금지: review-015가 D1~D5 독립 재컴파일 판정.
- Refs: directive `.claude/tree/gsts/directives/015-scaffold-codegen-fix.md`, contract `.claude/tree/gsts/nodes/fork-sync/workspace/contract-015-scaffold-fix.md`, spec `.../spec-impl-015.md`.

## Task #014 (signal-args 패치번들 — 완료 2026-06-06)
fork-sync~impl (executor): signal-args HIGH 패치 번들 + integration-workflow.md 초안 작성.
- 상태: **완료** (2026-06-06). TASK_RESULT 보고 예정.
- 산출물(모두 notes/ — src 미수정):
  - `notes/integration/patches/` — 20개 파일별 .patch (인벤토리 항목명으로 명명)
  - `notes/integration/patches/README.md` — 매핑표/재적용순서/완전성체크
  - `notes/integration-workflow.md` — 계약 6항목 워크플로
- 자가검수 금지(별도 review 노드 담당). 구현·문서화만 수행함.

## Refs
- 미션: `.claude/tree/gsts/directives/014-fork-sync-mission.md`
- 인벤토리(권위 분류표): `notes/fork-changes-inventory.md`
- 계약(출력 명세): `.claude/tree/gsts/nodes/fork-sync/workspace/contract-deliverables.md`
- 부모 스펙: `.claude/tree/gsts/nodes/fork-sync/workspace/spec-impl.md`
- 결과: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result.md`

## Notes
- 분기점 `71ca7a1` 선형. `git diff 71ca7a1 HEAD -- src/` = 20파일 +1022/-26 (계약/인벤토리와 일치 확인).
- 커밋 2개(66f4803 feat + 9415773 fix)가 그룹 A~E와 불일치 → format-patch 대신 **파일별 분기점 diff** 채택(계약 허용).
- **clean apply 검증 완료**: 임시 worktree(71ca7a1)에서 20패치 개별+일괄 `git apply --check` 전부 OK. worktree 제거·prune 완료. 워킹트리 src 미오염(untracked notes/integration/만).
- nodes.ts 특수: A8(sendSignal)+B5(parseValue)가 1파일 4hunk 공존, import 인접 → 물리 분리 시 독립 apply 불가 → 단일 파일패치 유지+이중 매핑. README에 hunk 명세.
- 버그수정 정적 확인: B1 varint guard 8건+, B2 any guard 1건, B5 entityLiteral 폴백 — 패치 내 보존 확인.
- D1(.gitignore)는 비코드 설정+.claude 관리외 → 번들 비포함, 추적표만 기록(의도적).
- 코드영향 22항목(A1~A13,B1~B5,C1~C6) 전부 캡처. B3/B4는 A2/A11 hunk 내포.
