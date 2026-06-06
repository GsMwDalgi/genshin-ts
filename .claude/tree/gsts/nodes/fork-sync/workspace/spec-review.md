# Task Spec — review 노드 (impl 산출물 독립 검수)

# Domain: fork-maintenance / independent-review
# Summary: implementation 노드가 만든 패치 번들 + 통합 워크플로 문서를, 우리 포크 변형이 빠짐없이 캡처됐는지·버그수정 무결성·충돌위험도 평가가 맞는지 **비판적·독립적**으로 검수한다. 구현하지 않음 — 검수만.

## 관점 (Org 제약 — 핵심)
- 너는 impl과 **별도 노드**다. impl이 자기 결과를 과대평가했을 가능성을 전제로, **누락·약화/롤백·오분류를 적극적으로 찾는다.** "맞겠지"가 아니라 "틀린 곳을 찾는다"는 관점.
- 너는 코드를 고치지 않는다. 검수 보고만 작성.

## 반드시 먼저 읽을 것 (Refs)
1. `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts\.claude\tree\gsts\directives\014-fork-sync-mission.md` — 미션.
2. `notes/fork-changes-inventory.md` — 권위 분류표 (이게 ground truth).
3. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-deliverables.md` — 산출물 계약 + 검수기준 C1~C5.
4. **impl 산출물** (검수 대상):
   - `notes/integration/patches/` (번들 + README)
   - `notes/integration-workflow.md`
5. 권위 델타 교차검증용: `git diff 71ca7a1 HEAD -- src/` (분기점 `71ca7a1`).

## 검수 항목 (계약 C1~C5 — 각 항목 pass/fail/partial + 근거)
- **C1 완전성**: 인벤토리 코드영향 22항목(A1~A13, B1~B5, C1~C6, D1)이 번들에 빠짐없이 캡처됐는가. 누락/중복 명시.
- **C2 버그수정 무결성** (치명적): B1(varint overflow guard), B2(any 타입가드), B5(parseValue entity fallback)가 정확히 포함되고 의미 보존됐는가. **무심코 약화/롤백/조건누락된 흔적이 있는지** 코드 hunk 수준에서 직접 대조.
- **C3 충돌위험도 정확성**: 워크플로/README의 HIGH/MED/LOW 분류가 인벤토리 및 실제 코드 변경 규모/핫스팟성과 일치하는가. 오분류 지적.
- **C4 재적용 가능성**: 번들이 `71ca7a1` 기준 깨끗하게 재적용 가능한 형태인가 (형식·순서·종속성). 가능하면 실제 검증 시도 (예: 임시 트리에서 `git apply --check` 또는 format-patch 적용 드라이런) — 단, **실제 repo 트리를 변형하지 말 것** (검사 전용 플래그/임시 워크트리 사용).
- **C5 워크플로 실행가능성**: 절차가 모호함 없이 따라할 수 있는가, proto 채택·버그체크리스트(B1/B2/B5)·검증법 누락 없는가.

## 산출물
- `notes/integration/review-audit.md` — 항목별 verdict(C1~C5) + 발견 이슈 목록(심각도) + 종합 판정(통과 / 조건부통과 / 반려) + 권고.

## 제약
- src/번들 수정 금지 — 검수 보고만. 워킹트리 더럽히지 말 것 (읽기/검사 전용).
- `.claude/`는 무시.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT로 보고. `review-audit.md` 경로 + 종합판정 한 줄 포함.
