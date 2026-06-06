# TASK_RESULT #015 — scaffold codegen 2 bug fix (fork-sync~impl)

## 수정 파일 (범위 = 1개)
- `src/cli/gil_scaffold.ts` — 그 외 src 미수정.

## 수정 내용 (정확한 라인/diff)

`buildDefaultValue()` 및 import 생성 로직. `git diff src/cli/gil_scaffold.ts` 요지:

### 버그1 — vec3 이중래핑 (D1)
reader(`injector/reader.ts` `decodeInitialValue`)는 vec3 initialValue로 **완성형** `vec3(x, y, z)` 문자열을 준다. 기존 코드는 ctor가 list/dict가 아니면 `${ctor}(${v.initialValue})`로 재래핑 → `vec3(vec3(0, 0, 0))`.
추가로 그 `vec3(x, y, z)`(3-positional-arg) 형식은 ambient global `vec3(value: Vec3Value): vec3`(인자 1개=배열)와 **arity 불일치**라 그 자체로 컴파일 불가(TS2554).

수정:
- 신규 헬퍼 `normalizeVec3(expr)`: `vec3(a, b, c)` → `vec3([a, b, c])` (이미 `vec3([...])`면 그대로). 단일 래핑 + 유효 arity 동시 충족.
- initialValue 경로: `if (ctor === 'vec3') return normalizeVec3(v.initialValue)` 추가 (재래핑 대신).
- fallback: `case 'vec3': return 'vec3(0, 0, 0)'` → `'vec3([0, 0, 0])'`.

### 버그2 — list/dict bare-comment (D2)
`entity_list` 등은 `TYPE_TO_CONSTRUCTOR`에서 ctor='list', reader가 initialValue를 주지 않음 → switch default 낙하 → `subParts: /* entity_list */,` (무효 TS, TS1109).

수정 (switch):
- `case 'dict': return 'dict(0)'` 추가 — ambient `dict(value: null|0): ReadonlyDict<never,never>` 유효 오버로드.
- default에 `*_list` 분기: `v.typeName.endsWith('_list')`면 elem = typeName 앞부분 → `return \`list('${elem}', [])\`` (ambient `list(type, items?)` 유효형, 빈 리스트 관례 = `list('t', [])`, tests/variables_definition_test.ts 확인).

### 추가 수정 (D4 달성에 필수) — import 무효 문제
기존 emit: `import { g, bool, float, int, prefabId, vec3 } from 'genshin-ts'`.
실측 결과 `g`/`bool`/`vec3`/`list`/`dict` 등은 **genshin-ts root의 named export가 아님** (package exports `.` = dist/src/index.js 런타임에 부재, types도 부재; `node -e` import 확인). 올바른 출처:
- `g` = `genshin-ts/runtime/core` named export.
- 값 생성자(bool/int/vec3/list/dict/...) = **ambient global** (gsts type package, `server_globals.d.ts` `declare global`). import 불필요 — tests도 import 없이 사용.

수정: `collectImports()` 제거, import emit을 상수 한 줄 `SCAFFOLD_IMPORT_LINE = "import { g } from 'genshin-ts/runtime/core'"`로 교체. 이 수정 없이는 어떤 변수 보유 scaffold도 tsc 불가(TS2305 다발) → 디렉티브 Goal("유효·컴파일 가능 TS")·D4가 충족 불가. 수정 범위는 동일 파일(gil_scaffold.ts) 내라 제약 준수.

## 검증 근거 (실제 실행, self-claim 아님)

도구: `npx tsx src/cli/gsts.ts scaffold ... ` (소스 CLI 직접, 빌드 불요) + `npx tsc`. 임시물은 R: 램디스크 및 in-repo `out/`(gitignore).

1. **회귀 .gil (D4)**: `1073741976.gil` 그래프 `AstroGear_05_Aegis`(id 1073741920, vec3 2개 + entity_list 1개 + 스칼라).
   - 수정 전 baseline tsc: `baseline.scaffold.ts(14,32): error TS1109: Expression expected` (버그2 무효 TS) — 재현 확인.
   - 수정 후 CLI 재생성(`final.scaffold.ts`, 수동편집 0) tsc: **스캐폴드 파일 에러 0건, FULL_EXIT=0**.
   - 생성 변수 블록(발췌):
     ```
     import { g } from 'genshin-ts/runtime/core'
     ...
       overPosition: vec3([0, 0, 0]),     // 버그1 해결: 단일·유효
       overRotation: vec3([0, 0, 0]),
       subParts: list('entity', []),      // 버그2 해결: 유효 빈 리스트
       skipFrame: int(0), bladePrefabID: prefabId(0), leavePilot: bool(1), pt: float(0),
     ```
2. **전 타입 분기 (D2/D3)**: scalar 10종(fallback) + initialValue-scalar 4종 + `*_list` 7종 + dict 를 망라한 합성 샘플 `typecheck-all.ts` → tsc **0 에러**. *_list/dict 모두 유효, 스칼라(initialValue 유무) 출력 형태 불변.
3. **src 무결성**: `tsc -p tsconfig.json --noEmit`에서 gil_scaffold.ts 관련 에러 0건.
4. **범위/오염 (D5)**: `git status --short src/` = `M src/cli/gil_scaffold.ts` 단 1개. `git check-ignore out/...` = ignored(루트 미오염). upstream 공유코드(compiler/runtime/definitions/proto) 미접촉. 다른 버그수정(varint guard 등) 미접촉.

## 자기평가 (D1~D5 self-claim — 최종판정은 review-015 독립 재컴파일)
- D1 vec3 단일래핑: **충족** — `vec3([0,0,0])` 단일·유효, 이중래핑 재현 안 됨.
- D2 list/dict 유효TS: **충족** — `list('entity',[])` / `dict(0)`, bare 주석 제거.
- D3 무회귀: **충족** — 스칼라·initialValue 경로 출력 불변, 합성샘플 전 분기 tsc 통과.
- D4 컴파일 실증: **충족(실증)** — 회귀 .gil scaffold tsc EXIT=0 (baseline은 TS1109 실패였음).
- D5 범위준수·루트미오염: **충족** — gil_scaffold.ts 단일 수정, out/ gitignore, R: 정리.

> 주의(review가 판단할 지점): import 무효 문제는 스펙 명시 2버그 외의 3번째 수정. D4 달성에 불가피하다고 판단해 동일 파일 내에서 수정했으나, "2버그 한정" 해석과 충돌 여지 — review가 이 추가수정의 타당성/범위 적정성을 함께 판정 요망. (자가검수 안 함.)
