# TASK_RESULT — fork-sync~impl (signal-args 패치 번들 + 통합 워크플로)

## 산출물 (모두 `notes/` 아래, src 미수정)

| 산출물 | 경로 |
|--------|------|
| 패치 번들 (20개 파일별 .patch) | `notes/integration/patches/` |
| 번들 매니페스트/매핑 README | `notes/integration/patches/README.md` |
| 통합 워크플로 문서 | `notes/integration-workflow.md` |

## 핵심 결정/사실

1. **형식**: 단일 선형 히스토리의 커밋 2개(`66f4803` feat + `9415773` fix)가 논리 그룹 A~E와 불일치 → `git format-patch`(커밋단위) 대신 **분기점 기준 파일별 diff**(`git diff 71ca7a1 HEAD -- <file>`)로 캡처. 계약이 명시 허용한 방식.
2. **재적용성 검증 완료**: 임시 worktree를 `71ca7a1`에 만들어 20개 패치를 `git apply --check`로 **개별·일괄 모두 clean apply 확인**. worktree 제거·prune 완료, 메인 워킹트리 src 미오염(untracked `notes/integration/`만 생성).
3. **권위 델타 일치**: `git diff 71ca7a1 HEAD -- src/` = 20파일 +1022/-26, 인벤토리/계약과 정확히 일치 확인.
4. **nodes.ts 이중매핑**: A8(sendSignal)+B5(parseValue)가 1파일 4hunk로 공존, import영역 인접 → 물리 hunk 분리 시 독립 apply 불가 → 단일 파일패치(`A8B5-...`) 유지 + 두 그룹 동시 매핑, README에 hunk 명세표 제공.
5. **D1(.gitignore)** 비코드 설정 + `.claude/` 관리외 → 번들 비포함, 워크플로 추적표에만 기록(의도적).

## 자기평가 (계약 C1~C5 — 단, 최종 판정은 review 노드)

- **C1 완전성**: 코드영향 22항목(A1~A13, B1~B5, C1~C6) 전부 패치에 캡처. B3/B4는 A2/A11 동일 hunk 내포, B5는 A8B5에 포함. D1은 비코드라 추적표만. → **충족(self-claim)**.
- **C2 버그수정 무결성**: B1(varint guard 3함수, grep 8건+), B2(any guard 1줄), B5(entityLiteral 폴백)를 패치에서 정적 확인, 약화/롤백 흔적 없음. → **충족(self-claim)**.
- **C3 충돌위험도**: 워크플로의 HIGH 7파일/MED/LOW 분류를 인벤토리 충돌표와 1:1 대조해 일치. → **충족(self-claim)**.
- **C4 재적용 가능성**: `71ca7a1` 기준 20패치 개별+일괄 `git apply --check` 통과(실증). → **충족(검증됨)**.
- **C5 워크플로 실행가능성**: 6항목(머지전략+분기점갱신 / HIGH 재적용순서+충돌해소 / proto 채택 / B1·B2·B5 롤백금지 체크리스트 / 빌드·타입체크·인젝션스모크 검증 / 추적표) 전부 포함, 명령·grep 검증식 구체화. → **충족(self-claim)**.

> 형식 메모: format-patch 미적용 사유 = 커밋경계≠그룹경계(위 #1). 대신 파일별 분기점 diff로 C4 실증.
> 자가검수는 수행하지 않음(Org 제약) — 위는 directive 기준 self-claim일 뿐, 누락·롤백·오분류 판정은 review 노드 몫.
