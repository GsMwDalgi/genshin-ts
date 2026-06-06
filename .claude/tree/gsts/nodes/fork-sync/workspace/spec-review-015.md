# Task Spec — review-015 노드 (scaffold fix 독립 검수)

# Domain: fork-maintenance / independent-review
# Summary: impl-015가 한 `src/cli/gil_scaffold.ts` 버그수정(vec3 이중래핑 / list·dict bare-comment)이 실제로 두 버그를 해결했고 회귀가 없는지 **비판적·독립적**으로 검수한다. 생성물을 직접 다시 컴파일해 검증. 구현 안 함 — 검수만.

## 관점 (Org 제약 — 핵심)
- 너는 impl-015와 **별도 노드**. impl이 자기 수정을 과대평가했을 가능성을 전제로, **회귀·범위이탈·미해결을 적극 적발**한다.
- 너는 코드를 고치지 않는다. 검수 보고만. 단, 검증을 위해 생성물 scaffold/tsc는 **직접 재실행**해 D4를 독립 재현한다.

## 반드시 먼저 읽을 것 (Refs)
1. `.claude\tree\gsts\directives\015-scaffold-codegen-fix.md` — 미션.
2. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-015-scaffold-fix.md` — 계약 + 검수기준 D1~D5.
3. impl 수정: `src/cli/gil_scaffold.ts` (`fork-sync~impl` 변경 후) + impl 보고 `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result.md`.
> 실행 노드 = 영속 `fork-sync~review` (너). 이건 #015 검수; #014 audit과는 별개 task.

## 검수 항목 (계약 D1~D5 — 각 pass/fail/partial + 근거)
- **D1 버그1 해결**: vec3 출력 단일래핑(이중래핑 재현 안 됨).
- **D2 버그2 해결**: *_list/dict 변수가 유효 TS 값 emit, bare 주석 없음.
- **D3 무회귀**: 스칼라 및 initialValue 경로 출력 불변·유효. 다른 버그수정(varint guard 등)·기존 동작 미손상 — diff를 직접 읽어 범위 밖 변경 없는지 확인.
- **D4 컴파일 실증 (독립 재현)**: 회귀 .gil(`...\1073741976.gil`, AstroGear_05_Aegis id 1073741920)을 **네가 직접 scaffold → tsc**. impl 결과를 신뢰하지 말고 재실행. 실제 tsc 출력/exit code를 근거로.
- **D5 범위 준수**: 수정이 gil_scaffold.ts 한정, upstream공유코드 미접촉, repo 루트 미오염(git status로 확인 — 생성물이 트리에 안 남았는지).
- **D6 (추가 — 반드시 판정) 3번째 수정(import 재작성)의 타당성·범위 적정성**: impl이 D4(tsc통과) 달성 위해 명시 2버그 외 **import emit 로직**도 수정함(`import {g,...} from 'genshin-ts'` → `import { g } from 'genshin-ts/runtime/core'` + collectImports 제거, 값 생성자는 ambient global이라 import 불필요라는 주장). 판정 사항:
  - (a) **사실 검증**: g가 정말 `genshin-ts/runtime/core` named export이고, bool/int/vec3/list/dict 등이 ambient global(import 불필요)인지 — package exports/types/`server_globals.d.ts` `declare global`/기존 tests 사용례를 **직접 확인**. impl 주장 신뢰 금지.
  - (b) **불가피성**: 이 수정 없이는 어떤 변수보유 scaffold도 tsc 불가(TS2305)라 디렉티브 Goal("유효·컴파일 가능 TS")·D4 충족 불가인지. 즉 in-goal한 3번째 결함 수정인가, 아니면 불필요/과잉인가.
  - (c) **부작용**: import 변경이 g 외 다른 생성자 사용 scaffold에 회귀를 일으키지 않는지(ambient global 미해결로 오히려 깨지는 경우 등).
  - fork-sync 판단: 동일파일·디렉티브 Goal 직결이라 **in-goal로 잠정 수용** 예정. review가 (a)(b)(c)로 독립 검증해 확정/반려.

## 검증 인프라
- 임시작업 R: 램디스크 사용. **repo 루트 더럽히지 말 것** (생성 .ts는 R: 또는 무시 경로).

## 산출물
- `notes/integration/scaffold-fix-audit.md` — 항목별 verdict(D1~D6) + 발견 이슈(심각도) + 종합 판정(통과 / 조건부통과 / 반려) + 권고. D4는 네 tsc 실행 근거 포함, D6은 (a)(b)(c) 판정 포함.

## 제약
- 코드 수정 금지 — 검수 보고만. 워킹트리 정리(생성물 잔존 시 지적).
- `.claude/`는 무시.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT. `scaffold-fix-audit.md` 경로 + 종합판정 한 줄.
