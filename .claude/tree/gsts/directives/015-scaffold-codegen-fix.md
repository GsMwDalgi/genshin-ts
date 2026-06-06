---
domain: fork-maintenance
---
# Directive #015
- Summary: Fix 2 scaffold codegen bugs in gil_scaffold.ts (vec3 double-wrap + list/dict bare-comment)
- Created: 2026-06-06

## Why
실제 .gil scaffold 검증(#013)에서 발견된 코드젠 버그 2개. 크래시는 아니지만 생성된 스캐폴드가 **컴파일 불가**해짐 — mwe가 이 기능을 의존할 예정이라 수정 필요. 이건 우리 포크 자산(C카테고리 CLI)이므로 upstream 충돌 없음.

## Bugs (둘 다 `src/cli/gil_scaffold.ts`, `buildDefaultValue()` 부근)
1. **vec3 이중 래핑** — 출력이 `overPosition: vec3(vec3(0, 0, 0))`. `buildDefaultValue()`가 `initialValue`를 ctor로 한 번 더 감싸는데, vec3의 reader `initialValue`는 이미 `vec3(0,0,0)` 형태 → 이중 래핑.
2. **list/dict 변수 무효 TS** — 출력이 `subParts: /* entity_list */,`. list 타입에 initialValue가 없으면 bare 주석(`/* entity_list */`)을 값 자리에 그대로 emit → 컴파일 실패.

## Goal
두 버그를 수정해, 어떤 변수 타입(vec3 / list / dict / 스칼라)이든 scaffold 출력이 **유효한 컴파일 가능 TS**가 되게 한다.

## Constraints (중요)
- **버그 수정 = 치명적, 신중하게.** 구현 노드와 리뷰 노드를 분리해 관점 전환 검수. 다른 버그 수정(varint guard 등)이나 기존 동작을 건드리지 말 것.
- 수정 범위는 `src/cli/gil_scaffold.ts`(+ 필요한 헬퍼)로 한정. upstream 공유 코드(compiler/runtime/definitions/proto) 미수정.
- 원본 형태 유지 원칙은 유지하되, 이 파일은 우리 고유 CLI라 자유롭게 수정 가능.

## Verification (필수 — 실제 실행)
- 회귀 케이스: `AstroGear_05_Aegis`(id 1073741920)는 vec3(`overPosition`/`overRotation`) + entity_list(`subParts`)를 모두 가짐. 이 그래프를 scaffold → 생성된 .ts가 **tsc로 컴파일 통과**하는지 확인.
- list/dict/vec3/스칼라 각 타입이 출력에 올바르게 나오는지 샘플 확인.
- 대상 .gil: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil`

## Output
- 코드 수정: `src/cli/gil_scaffold.ts`
- 리뷰 노드: 수정이 두 버그를 실제로 해결했는지 + 회귀 없는지 독립 검증 (생성물 컴파일 결과 근거 포함).
- 통합 verdict를 leader에 보고.

## Infra note
- 임시/분석 작업이 필요하면 **R: 드라이브(램디스크)** 사용 가능. 저장소 루트는 더럽히지 말 것.
