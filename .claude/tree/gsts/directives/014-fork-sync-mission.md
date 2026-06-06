---
domain: fork-maintenance
---
# Directive #014
- Summary: Standing mission — fork maintenance org (variant preservation + upstream deviation minimization + continuous integration)
- Created: 2026-06-06

## Mission (상시)
genshin-ts 포크의 장기 유지보수를 총괄한다. 핵심 4축:
1. **우리 개인 변형 유지** — signal-args 기능, 버그 수정, CLI 도구 등 우리만의 변경을 보존.
2. **코드 리뷰 & 검수** — 모든 통합/수정 결과를 독립적으로 검수.
3. **upstream 변형 최소화** — 원본 genshin-ts와의 차이를 최소화, 원본 형태를 최대한 유지하며 통합.
4. **통합 상태 지속 유지** — upstream 업데이트를 주기적으로 따라가며 머지 가능 상태 유지.

## Org 제약 (반드시 준수)
- **구현과 리뷰는 별도 노드로 분할** — LLM은 자기 결과물을 과대평가하는 경향이 있으므로, 구현 노드와 리뷰 노드를 분리하고 **관점을 바꿔** 검수한다. 같은 노드가 구현+자가검수 금지.
- **버그 수정은 치명적** — 우리 버그 수정(varint overflow guard, any 타입가드, parseValue entity fallback 등)은 신중하게 다룸. 통합 중 절대 무심코 되돌리지 말 것. 변경 시 반드시 리뷰 노드의 독립 검증을 거침.
- 모델은 opus 사용 가능 (토큰 아끼지 말 것 — 품질 우선).
- `.claude/` 디렉토리는 무시 (관리 대상 아님).
- 트리 깊이 2단계까지 사용 가능 — 필요한 구현/리뷰 자식 노드를 SPAWN_REQUEST로 요청.

## 기준 자료
- `notes/fork-changes-inventory.md` — 우리 포크 변경 전체 인벤토리 (충돌 위험도 포함). **필독.**
- 포크 분기점: upstream `71ca7a1` 위에 선형. `git diff 71ca7a1 HEAD`가 우리 변경의 권위 있는 델타.
- HIGH 충돌 핫스팟: signal-args 기능 7파일 (runtime/core.ts, definitions/nodes.ts, compiler/ir_to_gia_transform/{mappings,index,pins}.ts, injector/signal_nodes.ts).
- 우리는 .proto/gia_gen/thirdparty 미변경 → 병합 시 upstream proto 그대로 채택.

## First work item (upstream URL 없이 지금 가능한 준비 작업)
실제 머지는 원본 repo URL 확보 후 진행. 지금은 **통합 베이스라인 & 워크플로** 구축:
1. 구현 노드: signal-args HIGH 패치셋을 하나의 재적용 가능한 패치 번들로 정리 + 통합 워크플로/추적 문서 초안 (`notes/integration-workflow.md`).
2. 리뷰 노드: 그 번들과 추적 문서를 검수 — 우리 변형이 빠짐없이 캡처됐는지, 버그 수정 무결성, 충돌 위험도 평가가 맞는지 독립 검증.
3. 검수 통과분만 verdict로 묶어 보고.

## Output
- 자식 노드 결과를 종합한 통합 verdict + 산출물 경로를 leader에 TASK_RESULT로 보고.
- 산출물은 `notes/`에 (개인 문서 영역). 저장소 src는 이 First item 단계에서 수정하지 않음 (준비/문서화 단계).
