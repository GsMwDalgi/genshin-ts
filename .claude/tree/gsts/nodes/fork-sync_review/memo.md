# memo — fork-sync~review (영속 검수 노드)

## 정체성
- 노드명(tree-cli): `fork-sync~review` (tilde). spawn handle: `fork-sync-review` (hyphen).
- parent = fork-sync. 역할 = impl의 upstream 머지 **독립 검수**(impl≠review, Org 제약).
- 쓰기 허용: 이 노드 workspace/ + memo.md + (task 지정) `notes/integration/upstream-merge-audit.md`.

## Refs (다음 세대 필독)
- 계약: `.claude/tree/gsts/nodes/fork-sync/workspace/contract-017-upstream-merge.md`
- 내 task spec: `.claude/tree/gsts/nodes/fork-sync/workspace/spec-review-017.md`
- impl claim(신뢰금지): `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-017.md`
- 내 audit 산출: `notes/integration/upstream-merge-audit.md`

## #017 검수 완료 (2026-06-07) — VERDICT = PASS
검수 대상 상태: `integ-upstream` HEAD=9cb31c8(=upstream/master v0.1.10) + 워킹트리 B/C uncommitted. master=8629dc9 불변.

판정 요약:
- G0/R1 PASS: NEW_BASE=9cb31c8≠71ca7a1(ancestor 확인). backup tag·master 모두 8629dc9, force 없음.
- R2 PASS: A 잔재 grep 0(전체 src), upstream defineSignal/300000·300001 채택, diff=B/C만(10파일/832ins/6del).
- R3 PASS: B1 분산검증(signal_nodes 2 + binary 7 = 9, 3함수 가드 보존, diff로 fork추가물 확인), B2=1, B5=1. 롤백 없음.
- R4 PASS: install/tsc/build EXIT0. inspect/scaffold EXIT0(#015 픽스 보존). signal e2e 부분PASS(핵심 L164-188 통과, L190 fixture 게이트=우리무관). npm test EXIT1 = upstream 선존버그(pure upstream worktree 독립재현 동일).
- R5 폐기확정 PASS: 우리18종 ⊆ upstream SignalParamType(21항, +faction). 갭 없음(superset). _list 등가. C-하이브리드 불요. 미검증폐기 아님.
- R6 PASS: 마이그레이션 델타 정확. INFO: sendSignal 라인번호 7409-7411 표기 vs 실측 7414-7416(경미).

치명 0. 경미 2(비차단): signal e2e 핀 깊이검사 미완주(fixture 부재), R6 라인번호 드리프트.

## 검수 방법 메모 (재실행 시 함정 주의)
- ⚠️ worktree/batch 테스트는 **절대경로** 사용($PWD subshell cd 후 재평가 함정). 나는 `D:/MyDrive/Repos/_gsts_upstream_check`(upstream 재현)·`_gsts_scratch`(scaffold out) 절대경로 사용, 검수 후 제거+prune.
- 옛 18종 SIGNAL_ARG_TYPE_MAP은 `git show fork-backup-20260607:src/compiler/ir_to_gia_transform/mappings.ts`로 추출(현 트리엔 폐기됨).
- B1은 단일파일 grep로 판단 금지 — upstream이 readField*를 signal_nodes→binary.ts 이동. binary.ts+signal_nodes.ts 합산.
- npm test가 tests/generated·tests/enum_cases 재생성함 → 검수 후 `git checkout -- tests/` + untracked enum 제거로 워킹트리 복원.
- scaffold 단독 tsc는 ambient globals 미해소로 실패 정상(머지회귀 아님) — dist 대상 검증이 맞음.

## 다음 작업 대기
- TASK_RESULT(audit 포인터)를 leader에 송신 완료. parent가 verdict-017 통합 판단.
- 재작업 지시 시: leader 수용 단계에서 fixture 확보되면 signal e2e 300000/300001 핀 깊이검사 완주 권장(보강).
