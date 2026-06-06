# Task Spec — baseline C2 패치 refresh + 번들 일관성 (실행: fork-sync~impl)

> 실행 노드 = 영속 `fork-sync~impl` (너). 이건 #015 커밋(594a6e8) 후속 — #014 베이스라인 번들 갱신 작업. result는 너의 workspace/result-016.md에.

# Domain: fork-maintenance / baseline-refresh
# Summary: #015가 커밋(594a6e8)되어 `src/cli/gil_scaffold.ts`의 권위 델타가 236→251줄로 변함. #014 베이스라인 번들의 C2 패치를 신 델타로 refresh하고, 번들 20패치 전체가 여전히 `git diff 71ca7a1 HEAD`와 일치하는지 확인.

## 배경 (fork-sync 확인 완료)
- HEAD = `594a6e8 fix(scaffold): correct vec3 double-wrap and list/dict codegen` (#015).
- `git diff --stat 71ca7a1 HEAD -- src/cli/gil_scaffold.ts` = **251 insertions** (분기점 기준 전부 신규 파일이라 deletions 0; #015의 +31/-16이 누적되어 236→251).
- 기존 `notes/integration/patches/C2-cli-gil_scaffold.patch` = 236줄 스냅샷 → **stale**. 갱신 대상.
- 번들: `notes/integration/patches/*.patch` 20개.

## 반드시 먼저 읽을 것 (Refs)
1. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-deliverables.md` — #014 번들 계약 (패치 형식: `git diff 71ca7a1 HEAD -- <file>` 파일별 분기점 diff).
2. `notes/integration/patches/README.md` — 번들 매핑/형식.
3. `notes/integration/patches/C2-cli-gil_scaffold.patch` — 갱신 대상 현물.

## 해야 할 일
1. **C2 패치 refresh**: `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`로 신 델타(251줄) 재생성 → `notes/integration/patches/C2-cli-gil_scaffold.patch` 덮어쓰기. 기존 파일과 **동일 형식**(파일별 분기점 unified diff, 동일 헤더 스타일) 유지.
2. **번들 전체 일관성 확인**: 20패치 각각이 `git diff 71ca7a1 HEAD -- <해당파일>`과 **바이트 단위 일치**하는지 자동 대조(#014 review가 쓴 방식). C2 외 19패치는 #015로 안 바뀌었어야 정상 — 바뀐 게 있으면 보고.
3. **재적용성 재확인**: 갱신된 C2 포함 20패치가 분기점 `71ca7a1`에서 clean apply 되는지 임시 worktree에서 `git apply --check`(개별+일괄). **메인 워킹트리/트리 더럽히지 말 것** — worktree는 검사 후 remove+prune.
4. **README 갱신**(필요 시): C2 패치 줄 수/델타 언급이 있으면 251로 갱신. 매핑표 항목번호(C2) 불변.

## 제약
- 갱신 산출물은 `notes/integration/` 아래에만 (src 미수정 — 이건 베이스라인 문서 갱신).
- repo 루트/트리 미오염. 임시작업 R: 램디스크 가능.
- 자가검수 금지 — review가 독립 검증. 너는 갱신 + 자체 일관성 확인까지.

## 완료 기준 (review가 검수)
- E1: C2 패치 = 현 `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`와 바이트 일치(251줄 반영).
- E2: 번들 20패치 전부 권위 델타와 일치(C2 외 무변).
- E3: 분기점에서 20패치 clean apply(개별+일괄), 트리 무오염.
- E4: 형식/README 정합.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT. `result-016.md`(갱신 경로 + apply --check 근거 + 바이트대조 결과) + E1~E4 자기평가 한 줄(self-claim, 최종판정 review).
