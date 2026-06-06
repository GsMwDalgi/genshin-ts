# Task Spec — baseline refresh 독립 검수 (실행: fork-sync~review)

> 실행 노드 = 영속 `fork-sync~review` (너). #015 커밋 후 #014 베이스라인 C2 패치 refresh 검수.

# Domain: fork-maintenance / independent-review
# Summary: impl이 refresh한 C2 패치 + 번들 20패치 전체가 현 권위 델타(`git diff 71ca7a1 HEAD`)와 일치하고 분기점에서 clean apply 되는지 **독립 재현**으로 검증. 구현 안 함 — 검수만.

## 관점 (Org 제약)
- impl 자기주장 비신뢰. 바이트대조·apply --check를 **네가 직접 재실행**. 베이스라인 무결성이 핵심(이후 실제 머지의 기준 자료).

## 반드시 먼저 읽을 것 (Refs)
1. `.claude/tree/gsts/nodes/fork-sync/workspace/spec-impl-016-refresh.md` — impl 작업 명세(배경/E1~E4).
2. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-deliverables.md` — 번들 형식 계약.
3. impl 결과: `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-016.md` + 갱신된 `notes/integration/patches/`.

## 검수 항목 (E1~E4, 각 pass/fail/partial + 근거)
- **E1 C2 정확**: `notes/integration/patches/C2-cli-gil_scaffold.patch`가 `git diff 71ca7a1 HEAD -- src/cli/gil_scaffold.ts`(현 HEAD=594a6e8, 251줄)와 **바이트 일치**. 네가 직접 diff 재생성해 대조.
- **E2 번들 전체 일치**: 20패치 각각이 `git diff 71ca7a1 HEAD -- <파일>`과 바이트 일치(C2 외 19개는 #015로 무변이 정상 — 변동 있으면 적발).
- **E3 재적용**: 임시 worktree(71ca7a1)에서 20패치 `git apply --check` 개별+일괄 OK. 검사 후 worktree 제거·prune. **실제 repo 트리 무변형**(git status tracked 변경 0 확인 — notes/만 변경).
- **E4 형식/README**: 갱신 패치가 기존 형식 유지, README 정합(C2 줄수/매핑).

## 산출물
- `notes/integration/baseline-refresh-audit.md` — E1~E4 verdict + 이슈(심각도) + 종합판정(통과/조건부/반려). E1/E3는 네 실행 근거 포함.

## 제약
- 코드/패치 미수정 — 검수만. 워킹트리 정리(잔존 지적). `.claude/` 무시. R: 램디스크 가능.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT. `baseline-refresh-audit.md` 경로 + 종합판정 한 줄.
