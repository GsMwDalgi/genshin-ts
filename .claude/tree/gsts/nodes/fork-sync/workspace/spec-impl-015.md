# Task Spec — scaffold codegen 2 bug fix 구현 (실행: fork-sync~impl)

> 실행 노드 = 영속 `fork-sync~impl` (너). 이건 #015 작업; #014 패치번들과는 별개 task. result는 너의 workspace/result.md에 작성.

# Domain: fork-maintenance / scaffold-codegen-fix
# Summary: `src/cli/gil_scaffold.ts`의 코드젠 버그 2개(vec3 이중래핑 / list·dict bare-comment)를 수정해, 어떤 변수 타입이든 scaffold 출력이 유효·컴파일 가능한 TS가 되게 한다. 실제 실행으로 검증.

## 반드시 먼저 읽을 것 (Refs)
1. `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts\.claude\tree\gsts\directives\015-scaffold-codegen-fix.md` — 미션.
2. `.claude/tree/gsts/nodes/fork-sync/workspace/contract-015-scaffold-fix.md` — 산출물 계약 + 검수기준 D1~D5. **이게 출력 명세.**
3. 대상 코드: `src/cli/gil_scaffold.ts` `buildDefaultValue()` (현재 라인 52~72) + `TYPE_TO_CONSTRUCTOR`(라인 20~35).

## 버그 (fork-sync가 코드 확인, 그대로 사용)
1. **vec3 이중래핑** — 라인 56 `return \`${ctor}(${v.initialValue})\``. vec3의 `initialValue`가 이미 `vec3(0,0,0)` 완성형이라 재래핑 → `vec3(vec3(0,0,0))`.
2. **list/dict bare-comment** — 라인 70 `default: return \`/* ${v.typeName} */\``. `entity_list` 등(ctor='list'/'dict')에 `initialValue` 없으면 switch default 낙하 → `subParts: /* entity_list */,` (무효 TS).

## 해야 할 일
1. `src/cli/gil_scaffold.ts` 수정으로 두 버그 해결.
   - 버그1: vec3(및 ctor가 이미 완성형을 주는 타입)에서 이중래핑 제거. initialValue가 이미 ctor형이면 그대로, 아니면 한 번만 래핑 — 정확한 판단은 reader가 vec3 initialValue를 어떻게 주는지 코드로 확인 후 결정.
   - 버그2: list/dict(및 *_list)에 initialValue 없을 때 bare 주석 대신 **유효한 빈 컬렉션 TS**(예: `list()`/`dict()` 또는 프로젝트 관례에 맞는 형태) emit. 관례는 기존 코드/정의(definitions, runtime)에서 확인해 일치시킬 것.
2. **기존 동작 보존**: 스칼라 및 initialValue 있는 경로의 출력은 바뀌면 안 됨 (D3 무회귀).

## 검증 (필수 — self-claim 불가, 실제 실행)
- 회귀 .gil: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil` (그래프 `AstroGear_05_Aegis` id 1073741920 — vec3+entity_list 둘 다).
- scaffold 실행 → 생성된 .ts가 **tsc 컴파일 통과**하는지 확인. tsc 실제 출력/exit code를 result.md에 근거로 첨부.
- vec3 / *_list / dict / 스칼라 각 타입이 출력에 올바른 형태인지 샘플 확인.
- 빌드/실행 방법은 프로젝트 관례 확인(package.json scripts, gsts CLI 진입점 `src/cli/gsts.ts`의 scaffold 서브명령).

## 제약 (중요)
- 수정 범위 = `src/cli/gil_scaffold.ts`(+ 같은 파일 헬퍼)로 한정.
- **upstream 공유 코드(compiler/runtime/definitions/proto) 미수정.** 다른 버그수정(varint guard 등)·기존 동작 미접촉.
- **repo 루트(저장소 트리) 더럽히지 말 것** — 생성 .ts/임시물은 **R: 드라이브(램디스크)** 또는 무시 경로에. 커밋 안 함.
- 자기 결과 자가검수 금지 — 별도 review-015 노드가 검수·재컴파일. 너는 구현+자체 실행검증까지.

## 완료 기준 (review-015가 D1~D5로 검수)
- D1 vec3 단일래핑 / D2 list·dict 유효TS / D3 무회귀 / D4 회귀.gil scaffold가 tsc통과(실증) / D5 범위준수·루트미오염.

## 보고
완료 시 부모(fork-sync)에 TASK_RESULT. `result.md`(수정 요약 + tsc 근거 + 변경 diff 요지) 경로 + D1~D5 자기평가 한 줄 (self-claim, 최종판정은 review-015). 수정한 정확한 라인/diff를 result.md에 적을 것.
