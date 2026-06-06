# 독립 검수 보고 — Directive #015 (scaffold codegen fix)

> review 노드(fork-sync~review) 독립 검수. impl-015의 `src/cli/gil_scaffold.ts` 수정 대상.
> Ground truth: 디렉티브 #015, contract-015-scaffold-fix.md, 실제 코드 diff, 실제 tsc 재실행.
> 관점: 비판적·독립적. impl 자기주장(result-015.md) 비신뢰 — scaffold·tsc를 **직접 재실행**해 재현.
> 검수일: 2026-06-07. **워킹트리 무오염** (산출물은 R: 램디스크 + gitignored `out/`에만, 검수 후 전부 삭제. 최종 git status = `M src/cli/gil_scaffold.ts` 단 1건).

---

## 종합 판정: **통과 (PASS)** — D6 3번째 수정 포함 수용 권고

근거 한 줄: 두 명시 버그(vec3 이중래핑·list/dict bare-comment) + 3번째 수정(import emit)이 모두 해결됨을 **내가 직접 재생성한 scaffold의 tsc EXIT=0**으로 실증, **pre-fix baseline은 동일 파일에서 tsc 실패(TS1109)**로 인과 확인. 합성 전-분기 샘플도 tsc 통과. 범위는 gil_scaffold.ts 단일 파일, upstream 공유코드 미접촉, repo 루트 미오염. D6 3번째 수정은 (a)사실·(b)불가피·(c)무부작용 전부 독립 확인 → in-goal로 수용 권고.

---

## 항목별 verdict (D1~D6)

### D1 버그1 (vec3 이중래핑) 해결 — **PASS**
- 내가 직접 재생성한 회귀 scaffold 출력: `overPosition: vec3([0, 0, 0])`, `overRotation: vec3([0, 0, 0])` — 단일 래핑.
- pre-fix(HEAD) 재생성 출력: `overPosition: vec3(vec3(0, 0, 0))` — 이중래핑 재현 확인 → 수정으로 사라짐.
- 추가 검증: reader(`reader.ts:227`)가 vec3 initialValue를 **positional** `vec3(x, y, z)`로 emit. `normalizeVec3`가 이를 `vec3([x,y,z])`(array form)로 변환. `Vec3Value = vec3 | [number,number,number]`(value.d.ts:167)이고 ambient `vec3(value: Vec3Value)`는 단일 인자 → 구(舊) `vec3(0,0,0)` 3-positional은 arity 불일치(TS2554)였음. 신형은 단일래핑 + 유효 arity 동시 충족. impl 진단 정확.

### D2 버그2 (list/dict bare-comment) 해결 — **PASS**
- 회귀 출력: `subParts: list('entity', [])` — 유효 빈 타입드 리스트, bare 주석 없음.
- pre-fix 출력: `subParts: /* entity_list */,` 재현 → 수정으로 사라짐.
- ambient `list<T extends 'bool'|'config_id'|'entity'|'faction'|'float'|'guid'|'int'|'prefab_id'|'str'|'vec3'>(type: T, items?: ...[]|null|0)` (server_globals.global.d.ts:670+) 확인. impl이 `typeName.slice(0,-'_list'.length)`로 elem 추출 → TYPE_TO_CONSTRUCTOR의 7개 `*_list`(entity/int/float/str/bool/vec3/guid) 전부 union에 포함 → `list('<elem>', [])` 전부 유효.
- dict: ambient `dict(value: null|0): ReadonlyDict<never,never>` 오버로드(server_globals.global.d.ts:418) 확인 → `dict(0)` 유효.

### D3 무회귀 — **PASS**
- 합성 샘플(scalar 11종 fallback + 7 `*_list` + dict + vec3-initialValue 경로)을 내가 작성해 tsc → **EXIT=0**.
- diff 직접 정독: 변경은 `collectImports`→`SCAFFOLD_IMPORT_LINE` 교체, `normalizeVec3` 신규, vec3 initialValue 분기 추가, vec3 fallback `(0,0,0)`→`([0,0,0])`, dict/`*_list` case 추가, import emit 1줄. **스칼라 fallback(bool/int/float/str/guid/entity/prefab/config/faction) 및 비-list/dict initialValue 경로(`${ctor}(${v.initialValue})`)는 불변** → 회귀 없음.
- 다른 버그수정(varint guard 등) 및 타 파일 미접촉(diff가 gil_scaffold.ts 한정).

### D4 컴파일 실증 (독립 재현) — **PASS (실증)**
impl 결과 비신뢰, **내가 직접** 재실행:
1. `npx tsx src/cli/gsts.ts scaffold <1073741976.gil> --id 1073741920` → `regression.scaffold.ts` 생성(수동편집 0). vec3 2개 + entity_list + 스칼라 보유 그래프 `AstroGear_05_Aegis`.
2. repo tsconfig 상속한 tsconfig로 `tsc --noEmit` → **TSC_EXIT=0** (에러 0건).
3. **인과 증명**: HEAD(pre-fix) `gil_scaffold.ts`를 추출해 동일 .gil 재scaffold → `baseline.scaffold.ts`. tsc → **`baseline.scaffold.ts(14,32): error TS1109: Expression expected`, EXIT=2** (line 14 = `subParts: /* entity_list */,`). 즉 baseline 실패 ↔ fixed 통과 → 수정이 원인.
- 자료: 두 tsconfig 모두 프로젝트 tsconfig.json(`types:["gsts","node"]`, typeRoots `types-local`) 상속 → ambient global·self-package(`genshin-ts/runtime/core`) 해석을 tests와 동일 환경으로 재현.

### D5 범위 준수 — **PASS**
- diff 대상: `src/cli/gil_scaffold.ts` 단일. compiler/runtime/definitions/proto 등 upstream 공유코드 미접촉(git status 확인).
- repo 루트/트리 미오염: 모든 생성물은 R: 램디스크 + gitignored `out/review015`에만 생성, 검수 후 삭제. **최종 `git status --porcelain` (tracked) = `M src/cli/gil_scaffold.ts` 1건뿐.** (검수 중 임시로 src/cli에 둔 pre-fix 복사본 2개는 baseline 생성 직후 삭제 — 잔존 없음 확인.)

### D6 3번째 수정(import emit 재작성) 타당성 — **PASS (in-goal, 수용 권고)**
impl이 명시 2버그 외 import emit도 수정: `import {g, bool,...} from 'genshin-ts'` → `import { g } from 'genshin-ts/runtime/core'` + `collectImports` 제거.

- **(a) 사실 검증 — 전부 확인(impl 주장 독립 재확인):**
  - `g`는 `genshin-ts/runtime/core` named export ✔ — `src/runtime/core.ts:910 export const g`, 해석 .d.ts `dist/src/runtime/core.d.ts:233 export declare const g`. package.json exports `"./runtime/*": dist/src/runtime/*.{d.ts,js}` → 서브패스 유효. tests도 동일하게 `import { g } from 'genshin-ts/runtime/core'` 사용(`tests/variables_definition_test.ts:1`).
  - root `genshin-ts`(`.` export, types=`types/gsts/index.d.ts` → `export * from dist/src/index.js`)는 **bool/int/vec3/list/dict/g를 named export 안 함** ✔ — `dist/src/index.d.ts` 정독: compiler/injector/definitions 일부만 export, 값 생성자 0건. → 구 `import {g,bool,...} from 'genshin-ts'`는 TS2305 다발이 맞음(impl 진단 정확).
  - 값 생성자(bool/int/float/str/vec3/guid/entity/prefabId/configId/faction/list/dict)는 **ambient global** ✔ — `dist/src/runtime/server_globals.global.d.ts:28 declare global { function bool... vec3... list... dict... }`, root types가 triple-slash로 이를 참조. import 불필요. tests도 import 없이 사용.
- **(b) 불가피성 — 확인:** 이 수정 없이는 변수 보유 scaffold가 무조건 tsc 불가(구 import = TS2305). 디렉티브 Goal("어떤 변수 타입이든 유효·컴파일 가능 TS") 및 D4(tsc 통과)를 충족하려면 import 수정이 **필수**. baseline tsc는 bare-comment(TS1109)에서 먼저 멈췄지만, 그를 고쳐도 잘못된 import가 남으면 TS2305로 실패 → 2버그 수정만으로는 D4 미달. 따라서 in-goal한 3번째 결함 수정(과잉 아님).
- **(c) 부작용 — 없음:** import을 `{ g }` 단일로 축소해도 값 생성자는 ambient라 미해결 우려 없음. 합성 샘플(g 외 11 스칼라 + 7 list + dict + vec3 전부 ambient 사용)이 tsc EXIT=0 → ambient 해석 정상. 생성 출력에서 import 필요한 비-ambient 심볼은 `g`(=`g.server`)뿐임을 출력 스캔으로 확인. `collectImports` 잔존 참조 0건. 회귀 유발 경로 없음.
- **결론:** fork-sync의 "in-goal 잠정 수용"을 **확정 수용** 권고. 동일 파일·Goal 직결·불가피·무부작용.

---

## 발견 이슈 목록 (심각도)

| ID | 심각도 | 항목 | 내용 | 권고 |
|----|--------|------|------|------|
| (없음) | — | — | 회귀·범위이탈·미해결·오분류 발견 0건 | — |
| N1 | INFO (선택) | D5 시퀀싱 | #014 베이스라인 번들 `C2-cli-gil_scaffold.patch`는 #015 적용 전 스냅샷 → #015 머지 후 stale. (계약 시퀀싱 메모에 이미 인지됨) | #014 verdict에 "gil_scaffold.ts 베이스라인 refresh 필요" 기재(이미 계약에 명시) — #015 자체엔 영향 없음 |
| N2 | INFO (선택) | 견고성 | `normalizeVec3`는 `vec3(...)` 단일 매칭. reader가 항상 `vec3(x,y,z)`/`vec3([..])` 형태만 주므로 현재 안전. 향후 reader 출력 형식 변동 시 재확인 권장 | 현 시점 조치 불요 |

치명/중대 이슈: **0건.**

---

## 독립 검증 로그 (재현 가능)
1. D6(a) 정적: package.json exports / `dist/src/index.d.ts`(root named export) / `server_globals.global.d.ts`(declare global) / `core.d.ts`(g) / `value.d.ts`(Vec3Value) / tests 사용례 직접 확인.
2. D4: 내가 scaffold 재생성(EXIT=0) → tsc EXIT=0. pre-fix(HEAD) 재생성 → tsc TS1109 EXIT=2 (인과).
3. D3/D6(c): 합성 전-분기 샘플 tsc EXIT=0.
4. D5: git status tracked = `M gil_scaffold.ts` 1건, 임시 src 복사본 삭제 확인, out/·R: 산출물 삭제.

---

## 자기평가 (계약 D1~D6 기준)
- D1 vec3 단일래핑: **PASS** (재생성 출력 + pre-fix 대조).
- D2 list/dict 유효TS: **PASS** (재생성 출력 + ambient 시그니처 확인).
- D3 무회귀: **PASS** (합성 전-분기 tsc EXIT=0, 스칼라 경로 diff상 불변).
- D4 컴파일 실증(독립): **PASS(실증)** — fixed EXIT=0 vs baseline TS1109 EXIT=2.
- D5 범위/미오염: **PASS** — gil_scaffold.ts 단일, 공유코드 미접촉, tracked 1건.
- D6 import 수정 타당성: **PASS** — (a)사실·(b)불가피·(c)무부작용 전부 독립 확인, in-goal 수용 권고.
- 제약 준수: 코드 미수정(검수만), `.claude/` 무시, 워킹트리 정리(잔존 0). **준수.**
- 한계: 검증 환경 tsc는 프로젝트 tsconfig 상속본 사용(tests와 동일 환경) — 빌드 산출물(dist/types-local) 기준. 런타임(실제 인젝션) 동작은 본 검수 범위 밖(디렉티브가 요구한 건 "컴파일 가능 TS"이며 그건 충족).

**종합: 통과 (PASS) — 조건 없음. D6 3번째 수정 확정 수용 권고. N1/N2는 INFO(차단 아님).**
