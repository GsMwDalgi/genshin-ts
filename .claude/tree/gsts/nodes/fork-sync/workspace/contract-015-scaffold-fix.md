# 산출물 계약 — Directive #015 (scaffold codegen 2 bug fix)

impl 노드와 review 노드가 공유하는 단일 진실. 두 노드 모두 따른다.
> 실행 노드 = **영속 `fork-sync~impl` / `fork-sync~review`** (per-directive 신규 노드 아님 — leader 설계: 영속 쌍을 모든 fork-maintenance 디렉티브에 재사용). 본 문서의 "impl-015/review-015"는 역할 라벨일 뿐, 실제 노드는 그 둘. 산출물 result.md는 각 노드 자신의 workspace/에 작성.

## 대상 / 범위
- 수정 파일: `src/cli/gil_scaffold.ts` (+ 필요 시 같은 파일 내 헬퍼). 그 외 src 미수정.
- 우리 고유 CLI(C카테고리) → upstream 충돌 없음. 자유 수정 가능하되 **기존 동작/다른 버그수정(varint guard 등) 미접촉**.

## 버그 (확인됨 — fork-sync가 코드 직접 확인, 2026-06-06)
파일 `buildDefaultValue()` (현재 src 라인 52~72 부근):
1. **vec3 이중래핑** — 라인 56 `return \`${ctor}(${v.initialValue})\``. vec3의 reader `initialValue`가 이미 `vec3(0,0,0)` 완성형 → 재래핑 시 `vec3(vec3(0,0,0))`.
   - 출력 증상: `overPosition: vec3(vec3(0, 0, 0))`.
2. **list/dict bare-comment** — 라인 70 `default: return \`/* ${v.typeName} */\``. `entity_list`(라인28에서 ctor='list'로 매핑)에 `initialValue` 없으면 switch에 해당 case 없어 default 낙하 → 값 자리에 bare 주석.
   - 출력 증상: `subParts: /* entity_list */,` (무효 TS, 컴파일 실패).

## Goal
어떤 변수 타입(vec3 / *_list / dict / 스칼라)이든 scaffold 출력이 **유효·컴파일 가능한 TS**.

## 산출물 위치
| 산출물 | 경로 | 작성자 |
|--------|------|--------|
| 코드 수정 | `src/cli/gil_scaffold.ts` | impl-015 |
| 수정 보고 + 검증 근거 | `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result.md` | fork-sync~impl |
| 독립 검수 보고 | `notes/integration/scaffold-fix-audit.md` | review-015 |

## 검증 (필수 — 실제 실행, self-claim 불가)
- 회귀 .gil: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil`
  - 그래프 `AstroGear_05_Aegis` (id 1073741920): vec3(`overPosition`/`overRotation`) + entity_list(`subParts`) 둘 다 보유 → 두 버그 동시 회귀 케이스.
- 절차: 이 .gil을 scaffold → 생성된 .ts가 **tsc로 컴파일 통과**하는지 확인 (스캐폴드 산출물 + 그 산출물이 의존하는 타입까지 타입체크).
- 타입별 샘플 확인: vec3 / *_list / dict / 스칼라가 출력에 올바른 형태로 나오는지.
- 임시작업 R: 램디스크 사용 가능. **repo 루트(저장소 트리) 더럽히지 말 것** — 생성 .ts/임시물은 R: 또는 무시 경로에.

## 검수 기준 (review-015가 판정, fork-sync가 채택)
- **D1 버그1 해결**: vec3 출력이 단일 래핑(`vec3(0, 0, 0)` 또는 올바른 값). 이중래핑 재현 안 됨.
- **D2 버그2 해결**: *_list/dict 변수가 유효 TS 값(예: 빈 list/dict ctor) emit, bare 주석 없음.
- **D3 무회귀**: 스칼라(bool/int/float/str/guid/entity/...) 및 initialValue 있는 경우 출력이 이전과 동일하게 유효. 다른 버그수정/동작 미손상.
- **D4 컴파일 실증**: 회귀 .gil scaffold 산출물이 tsc 통과 (근거: 실제 tsc 출력/exit code).
- **D5 범위 준수**: 수정이 gil_scaffold.ts에 한정, upstream공유코드 미접촉, repo 루트 미오염.

## 관점 분리 (Org 제약)
- impl-015는 자기 수정을 검수하지 않는다. review-015는 별도 노드로 회귀·범위이탈을 비판적으로 찾고, **생성물을 직접 다시 컴파일**해 D4를 독립 재현한다.

## 시퀀싱 메모 (fork-sync 관리)
- #014 베이스라인 번들의 `C2-cli-gil_scaffold.patch`는 #015 적용 전 스냅샷. #015 머지 후 그 패치는 stale → #014 verdict에 "gil_scaffold.ts 베이스라인 refresh 필요" 기재. (#015 자체 산출물엔 영향 없음.)
