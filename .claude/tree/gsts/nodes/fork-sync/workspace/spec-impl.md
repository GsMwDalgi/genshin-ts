# Task Spec — implementation 노드 (signal-args 패치 번들 + 통합 워크플로)

# Domain: fork-maintenance / integration-baseline
# Summary: 우리 포크의 signal-args HIGH 패치셋을 재적용 가능한 패치 번들로 정리하고, upstream 머지용 통합 워크플로/추적 문서를 작성한다. src 미수정 — 산출물은 `notes/`에만.

## 배경
genshin-ts 포크 장기 유지보수 준비 단계. upstream repo URL은 아직 없음 → 실제 머지는 나중. 지금은 머지가 가능해지면 즉시 따를 수 있는 **통합 베이스라인 & 워크플로**를 만든다.

## 반드시 먼저 읽을 것 (Refs)
1. `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts\.claude\tree\gsts\directives\014-fork-sync-mission.md` — 미션 (특히 First work item).
2. `notes/fork-changes-inventory.md` — 포크 변경 전체 인벤토리 + 충돌 위험도. **권위 분류표.**
3. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-deliverables.md` — 산출물 계약 (경로/형식/검수기준). **이게 너의 출력 명세다.**

## 권위 델타
- 분기점: upstream `71ca7a1` (선형). `git diff 71ca7a1 HEAD -- src/` = 권위 델타 (20 src 파일, +1022/-26, 인벤토리와 일치 확인됨).

## 해야 할 일
1. **패치 번들 작성** → `notes/integration/patches/`
   - 계약의 그룹 A~E 분할대로 캡처. HIGH 시그널-인자 기능 + 종속 변경을 재적용 단위로 정리.
   - 형식: `git format-patch` (분기점 `71ca7a1` 기준 논리 그룹) 권장. 단일 선형 히스토리라 커밋 경계가 그룹과 안 맞으면, **명시적 hunk 문서 + 파일별 diff**로 대체 가능 — 핵심은 "분기점에서 깨끗하게 재적용 가능"한 형태.
   - `notes/integration/patches/README.md`: 각 번들 파일 ↔ 변경 그룹 ↔ 인벤토리 항목번호(Axx/Bxx/Cxx/D1) 매핑 표. 재적용 순서·종속성 명시.
2. **통합 워크플로 문서** → `notes/integration-workflow.md` (계약 형식 6항목 충족)
   - 머지 전략, HIGH 7지점 재적용 순서/충돌 해소, proto는 upstream 채택 명시, **버그수정 무결성 체크리스트(B1/B2/B5 롤백금지)**, 재적용 후 검증법, 추적 표.

## 제약 (필독)
- **src 수정 금지** — 준비/문서화 단계. 모든 산출물은 `notes/` 아래.
- **버그수정 절대 약화/롤백 금지**: B1(varint overflow guard, signal_nodes.ts), B2(any 타입가드, ts_type_utils.ts), B5(parseValue entity fallback, nodes.ts). 번들에 정확히 보존.
- `.claude/`는 무시 (관리 대상 아님).
- 자기 결과를 자가검수하지 말 것 — 별도 review 노드가 검수한다. 너는 구현·문서화만.
- 작업은 패치 추출/문서화이므로 git 워킹트리를 더럽히지 말 것 (필요 시 `git format-patch`/`git diff`는 읽기 전용).

## 완료 기준 (review 노드가 이 기준으로 검수한다 — 계약 C1~C5)
- C1 완전성: 인벤토리 코드영향 22항목 빠짐없이 캡처.
- C2 버그수정 무결성: B1/B2/B5 정확 포함·의미 보존.
- C3 충돌위험도 정확성: 워크플로의 HIGH/MED/LOW가 인벤토리·코드와 일치.
- C4 재적용 가능성: 번들이 `71ca7a1` 기준 깨끗 재적용 가능 형태.
- C5 워크플로 실행가능성: 절차 모호함 없음, proto채택·버그체크리스트 포함.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT로 보고. 산출물 경로 + 계약 C1~C5에 대한 자기평가 한 줄 포함 (예: "C1~C5 met; C4는 hunk문서 방식 — format-patch 미적용 사유 X"). "완료"만으로는 수용 아님.
