# 산출물 계약 (Deliverables Contract) — First work item / Directive #014

implementation 노드와 review 노드가 공유하는 단일 진실. 두 노드 모두 이 계약을 따른다.

## 분기점 / 권위 델타
- 분기점: upstream `71ca7a1` (선형). 권위 델타 = `git diff 71ca7a1 HEAD -- src/` (20 src 파일, +1022/-26).
- 확인됨(2026-06-06): stat이 `notes/fork-changes-inventory.md`와 정확히 일치. 인벤토리가 권위 분류표.

## 산출물 위치 (둘 다 `notes/` 아래 — src 미수정)
| 산출물 | 경로 | 작성자 |
|--------|------|--------|
| 패치 번들 디렉터리 | `notes/integration/patches/` | impl |
| 통합 워크플로/추적 문서 | `notes/integration-workflow.md` | impl |
| 번들 README/매니페스트 | `notes/integration/patches/README.md` | impl |
| 독립 검수 보고 | `notes/integration/review-audit.md` | review |

## 패치 번들 형식 (impl)
- 시그널-인자 기능 HIGH 7지점 + 종속 변경을 **재적용 가능한 단위**로 분할 캡처.
- 권장: 논리 그룹별 `git format-patch` 또는 명시적 hunk 문서. 어느 쪽이든 **`71ca7a1`에서 깨끗하게 분기점 기준 재적용 가능**해야 함.
  - 그룹 A (signal-args core): runtime/{value,core,ir_builder,meta_call_types,IR.d,server_globals,server_globals.d}.ts, definitions/{nodes(sendSignal),events-payload}.ts
  - 그룹 B (compiler): ir_to_gia_transform/{mappings,index,pins}.ts
  - 그룹 C (injector signal): injector/signal_nodes.ts (signal-args A13 + 버그 B1 varint guard)
  - 그룹 D (bug fixes 독립): shared/ts_type_utils.ts (B2 any guard), definitions/nodes.ts parseValue hunk (B5 entity fallback)
  - 그룹 E (CLI/Reader, 충돌 LOW): cli/{gil_inspect,gil_scaffold,gsts}.ts, injector/reader.ts, i18n/*.json — 별도 번들 (HIGH와 분리)
- 각 그룹: 파일·hunk·인벤토리 항목번호(Axx/Bxx/Cxx) 매핑을 README에 명시.

## 통합 워크플로 문서 형식 (impl)
- upstream URL 확보 후 실제 머지 시 따를 절차. 최소 포함:
  1. 머지 전략 (rebase-on-upstream vs merge), 분기점 갱신 절차.
  2. HIGH 7지점 재적용 순서 + 충돌 발생 시 해소 가이드.
  3. proto는 upstream 채택 (우리 미변경) 명시.
  4. 버그 수정 무결성 체크리스트 (B1/B2/B5 절대 롤백 금지 — 검증 항목).
  5. 재적용 후 검증 방법 (빌드/타입체크/인젝션 스모크).
  6. 추적 표: 각 변경 그룹의 상태(미적용/적용/검증완료).

## 검수 기준 (review가 판정, fork-sync가 채택)
- **C1 완전성**: 인벤토리의 코드 영향 22항목(A1~A13, B1~B5, C1~C6, D1)이 번들에 빠짐없이 캡처됐는가. 누락 항목 명시.
- **C2 버그수정 무결성**: B1(varint overflow guard), B2(any 타입가드), B5(parseValue entity fallback)가 번들에 **정확히** 포함되고 의미 보존됐는가. 무심코 약화/롤백된 흔적 없는가.
- **C3 충돌위험도 정확성**: 워크플로 문서의 HIGH/MED/LOW 분류가 인벤토리 및 실제 코드와 일치하는가. 잘못 평가된 항목 지적.
- **C4 재적용 가능성**: 번들이 `71ca7a1` 기준 깨끗하게 재적용 가능한 형태인가 (형식/순서/종속성 측면).
- **C5 워크플로 실행가능성**: 절차가 모호함 없이 따라할 수 있는가, proto 채택·버그체크리스트 누락 없는가.
- review는 **구현하지 않음** — 검수만. verdict는 항목별 pass/fail/partial + 근거.

## 관점 분리 (Org 제약)
- impl은 자기 결과를 검수하지 않는다. review는 별도 노드로, **누락·롤백·오분류를 찾는 비판적 관점**으로 본다.
