# Result — Task #017 Upstream Merge (impl, 방향 B 실행)

> 실행 노드: fork-sync~impl. 방향 = **B (group A 폐기 + upstream defineSignal 채택)**, leader+user 확정.
> 진행 상세 로그: `workspace/merge-log-017.md`. 이전 발견(escalate)記: `workspace/escalation-017.md`.
> 자가검수 금지 — 본 result는 impl self-claim + 관찰 증거. 독립 검수는 fork-sync~review.
> **상태: integ-upstream 브랜치 = upstream/master(9cb31c8) + 워킹트리 B/C 변경 (uncommitted). master 불변. 커밋·master-update는 leader 수용 후 별도 단계.**

## 접근 (방식 선택 근거)
- spec REV2 권장 = "베이스 교체 + 선별 재적용". rebase 방식은 첫 커밋(66f4803=group A 포함)에서 HIGH-7 전수 충돌 → A를 한 hunk씩 폐기하며 B/C만 살리는 작업이 오류 유발적.
- 채택: `git reset --hard upstream/master`로 integ-upstream을 깨끗한 upstream v0.1.10으로 만들고, **유지대상(B1/B2/B5 + C)만 upstream 새 구조 위에 수동 재이식**. group A(A1~A13)는 단 한 줄도 재적용 안 함 → upstream defineSignal이 대체.
- 결과 검증: `git diff --stat upstream/master -- src/` = **B/C 변경만**(아래 G2 표). group A 코드 부재 확정.

---

## G0 — NEW_BASE 판정 (게이트) — **met**
- `git fetch upstream` (URL `https://github.com/josStorer/genshin-ts`), 기본브랜치 = `master`.
- **NEW_BASE = `9cb31c8`** (release v0.1.10). 우리 분기점 `71ca7a1`.
- NEW_BASE != 71ca7a1 → upstream 전진(`71ca7a1..upstream/master` = 43 커밋, 71ca7a1은 진짜 ancestor) → **머지 진행 게이트 통과**.
- 근거: `git rev-parse upstream/master` = 9cb31c8; `git merge-base --is-ancestor 71ca7a1 upstream/master` = true.

## G1 — 브랜치+태그 가역성 — **met**
- 백업 태그: **`fork-backup-20260607`** = `8629dc9` (`git rev-parse fork-backup-20260607` 확인).
- 작업 브랜치: **`integ-upstream`** (HEAD = 9cb31c8 + uncommitted B/C).
- master 불변: `git rev-parse master` = **`8629dc9`** (force·이동 안 함).
- 가역: 모든 작업이 integ-upstream + 백업태그로 복원 가능. master·upstream 무손상.

## G2 — HIGH-7 + B/C 처리 (방향 B: A폐기·upstream채택·B/C재이식) — **met**

| 파일 | 처리 | A 폐기 | B/C 재이식 |
|------|------|--------|------------|
| `runtime/value.ts` | upstream 채택 그대로 | A1(SignalArgDef/SignalArgsToPayload) 폐기 — upstream SignalParamType 등이 대체 | — |
| `runtime/core.ts` | upstream `onSignal<S extends SignalDefinition>`/`defineSignal` 채택 | A2(onSignal<Args>)·B3 폐기 | — |
| `definitions/nodes.ts` | upstream sendSignal 채택(A8 폐기) + **B5 재이식** | A8 폐기 | **B5**: parseValue `case 'entity'`에 `z.union([z.int(),z.bigint()])`→`new entityLiteral(result.data)` 폴백 추가 + `entityLiteral` import 추가(value.js). upstream `entityLiteral`(runtime/value.ts:309) 사용. |
| `compiler/.../mappings.ts` | upstream `send_signal:300000`/`monitor_signal:300001` 채택 | A10(SIGNAL_ARG_TYPE_MAP) 폐기 | — |
| `compiler/.../index.ts` | upstream 핀생성 루프 채택(L324-356) | A11·B4 폐기 | — |
| `compiler/.../pins.ts` | upstream 채택 | A12(ensureInputPinWithType) 폐기 | — |
| `injector/signal_nodes.ts` | upstream 채택(A13 폐기) + **B1 부분 재이식** | A13 폐기 | **B1**: `parseNodeGraphId`에 `offset>=0` 루프가드 + wire2 분기 `newOffset<0` break. (readField*는 upstream이 binary.ts로 이동 → 아래.) |
| `injector/binary.ts` | upstream 채택 + **B1 주이식** | — | **B1**: upstream의 `readFieldBytes`/`readFieldMessages`에 `offset>=0` 루프가드 + `len<0` break + `dataEnd<0` break 추가. |
| `shared/ts_type_utils.ts` | upstream 채택 + **B2 재이식** | — | **B2**: `isEntityLikeType` 본문 첫줄 `if (type.flags & ts.TypeFlags.Any) return false`. |
| `cli/gil_inspect.ts`, `cli/gil_scaffold.ts`, `injector/reader.ts` | **C 신규파일 추가**(upstream 미보유, 충돌0) | — | **C**: 8629dc9에서 그대로. gil_scaffold는 #015 픽스 포함(vec3([])/list()/dict()/`import{g}from 'genshin-ts/runtime/core'`). |
| `cli/gsts.ts` | upstream 채택 + **C 서브커맨드 등록 재이식** | — | **C**: `inspect`/`scaffold` `.command()` 블록(open과 help 사이) + `runInspect`/`runScaffold` import. upstream `gil_signals.ts`는 다른 파일이라 충돌 없음. |
| `i18n/locales/{en-US,zh-CN}/main.json` | upstream 채택 + **C i18n 키 재이식** | — | **C**: inspect/scaffold용 10키(cmdInspect/Scaffold, inspect*·scaffold* 옵션) 각 로케일. JSON 유효성 검증 OK. |
| proto/gia_gen/thirdparty (STEP3) | **upstream 그대로** — 우리 변경 0, 충돌 0 | — | — |

**src diff vs upstream/master 최종**(`git diff --stat upstream/master -- src/`): 10 파일, 832 insertions / 6 deletions — 전부 B/C. group A 코드 0.
- 신규(A): gil_inspect.ts(+153), gil_scaffold.ts(+251), reader.ts(+367).
- 수정(M): gsts.ts(+24), nodes.ts(+5), en-US(+10), zh-CN(+10), binary.ts(+10/-6), signal_nodes.ts(+6/-2), ts_type_utils.ts(+2).

### ★ review 주의 1 — B1 가드 위치 분산 (upstream 구조변경)
upstream v0.1.10이 `readFieldBytes`/`readFieldMessages`/`readFieldVarint`를 **signal_nodes.ts에서 binary.ts로 이동**함(우리 8629dc9에선 signal_nodes.ts 로컬 함수였음). 따라서 B1 가드를 원위치 따라 분산 재이식:
- `binary.ts`: readFieldBytes + readFieldMessages 2함수에 가드 (grep 7건; 추가로 upstream 자체의 parseMessage에 기존 `dataEnd>end||len<0` 1건이 grep에 잡힘).
- `signal_nodes.ts`: parseNodeGraphId 1함수에 가드 (grep 2건).
→ **spec §4 grep을 signal_nodes.ts 단독으로 보면 2건**(스펙의 "8건 이상" 기대와 불일치) — 이는 가드 누락이 아니라 **upstream이 함수를 옮긴 결과**. binary.ts 합산 시 가드 의미(while 음수가드/len<0/dataEnd<0/newOffset<0)는 3함수 전부 보존됨. review는 binary.ts+signal_nodes.ts 합산으로 확인 요망.

## G3 — B1/B2/B5 무결성 (§4, 절대 롤백금지) — **met (위치분산 주석)**
impl 실행 grep 카운트:
| 버그fix | grep | 카운트 | 비고 |
|---------|------|--------|------|
| B1 | `grep -nE "offset >= 0\|len < 0\|dataEnd < 0\|newOffset < 0" src/injector/signal_nodes.ts` | **2** | parseNodeGraphId만 남음(나머지 함수 binary.ts로 이동) |
| B1(분산) | 동 패턴 `src/injector/binary.ts` | **7** | readFieldBytes+readFieldMessages 가드 + upstream parseMessage 기존가드 |
| B1 합계 | signal_nodes + binary | **9** | 3함수 전부 가드 존재(의미보존) |
| B2 | `grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts` | **1** | isEntityLikeType 첫줄 |
| B5 | `grep -n "new entityLiteral" src/definitions/nodes.ts` | **1** | parseValue case 'entity' 폴백 |

## G4 — §5 검증 (구체 증거 + EXIT) — **met (npm test는 upstream 선존버그로 1, 우리무관)**
| 단계 | 명령 | EXIT | 결과 |
|------|------|------|------|
| install | `npm install` | **0** | 15 packages changed(upstream dep 갱신 반영), 0 vuln |
| typecheck | `npx tsc --noEmit` | **0** | 클린(B/C가 upstream v0.1.10에 정합 컴파일) |
| build | `npm run build` (= `tsc -p tsconfig.json` + postbuild) | **0** | dist 생성, proto/d.ts 복사 OK |
| §4 grep | (위 G3) | — | B1=9(분산)/B2=1/B5=1 |
| smoke: scaffold tsc | 생성 scaffold를 dist 대상 tsc | **0** | #015 회귀와 동일 — 생성 TS 0에러 |
| smoke: CLI inspect | `gsts inspect 1073741976.gil` | **0** | 184 node graphs 정상 나열(5.4MB, B1 무한루프 없음) |
| smoke: CLI scaffold | `gsts scaffold ... --id 1073741920` | **0** | AstroGear_05_Aegis.scaffold.ts 생성(vec3([])/list('entity',[])/`import{g}` 픽스 보존) |
| smoke: signal-args e2e | `npx tsx scripts/assert-signal-parameters.ts` | 1(부분) | **핵심 PASS**: defineSignal→GS 컴파일 + typed `evt.params.参数_1/2/3` + sendSignal 어서션(L174-188) 통과. L190에서 로컬 fixture `src/resources/signals.ts`(upstream 미커밋, gitignore 아님) 부재로 중단 — **우리 변경과 무관**(스크립트·픽스처 우리 미수정). |
| full suite | `npm test` | 1 | `other.literal.ts:11 "cannot infer list type"` 에서 실패. **PURE upstream/master worktree에서도 동일 실패 재현** → **upstream v0.1.10 자체 선존버그, 우리 머지와 무관**(증거: temp worktree `git worktree add upstream/master` → npm install → npm test = 동일 EXIT1·동일 메시지). |

- signal smoke 보충: `assert-signal-parameters.ts`는 upstream이 커밋한 정식 e2e(`tests/signal_parameters_test.ts` = defineSignal 3종 + typed evt.params + sendSignal). 우리 머지 트리에서 GS 변환·핀 어서션 전단계가 통과 = **방향 B(upstream signal-args)가 우리 트리에서 동작함**을 실증. (300000/300001 핀 깊이검사는 L190 fixture 게이트 뒤라 미도달 — review가 fixture 확보 시 재실행 권장; 또는 우리 .gil로 signals 추출 후.)

## G5 — 변형폐기 결정 (방향 B 핵심) — **방향 B 실행함 / 동등성 superset 관찰 (폐기확정은 review)**
- **폐기한 group A**: A1(SignalArgDef/SignalArgsToPayload, value.ts), A2(onSignal<Args>, core.ts), A8(sendSignal signalArgs, nodes.ts), A10(SIGNAL_ARG_TYPE_MAP 18종, mappings.ts), A11(send/monitor 핀생성, index.ts), A12(ensureInputPinWithType, pins.ts), A13(ClientExec 우선탐색, signal_nodes.ts). + 내포 B3(A2)/B4(A11).
- **대응 upstream 등가**: `defineSignal()`/`SignalDefinition`/`onSignal<S>` typed `evt.params`(core.ts) + `send_signal:300000`/`monitor_signal:300001` + per-param 핀생성(index.ts L324-356) + `gil_signals.ts` 추출.
- **동등성 관찰 (부록 기준)**:
  - **18종 타입 커버리지**: upstream `SignalParamType`(core.ts) = 우리 A10 18종(9스칼라+9 _list, vec3/vec3_list 포함)을 **전부 포함** + `faction`/`faction_list` 추가. 즉 upstream ⊇ 우리 = **갭 없음(superset)**. (set 비교: ours\upstream=∅, upstream\ours={faction,faction_list}.)
  - **`_list` 자동래핑**: upstream `tests/signal_parameters_test.ts` + `assert-signal-parameters.ts`(L192-210)가 `float_list`/`str_list`/`vec3_list`/.../`config_id_list` 추출·정의를 어서션 → upstream이 _list 처리 보유. (단 추출 fixture 게이트로 우리 환경 실행은 부분 — review 독립확인 대상.)
  - **결론(잠정→review)**: 관찰상 동등성 갭 없음 → C-하이브리드 보강 불요. **단 폐기 확정은 Org 제약상 review 독립검증 후**. impl은 "방향 B 실행 + superset 관찰" 보고, review 동등성 게이트 대기.
- C-하이브리드 보강: **없음**(갭 미관찰).

### ★ 마이그레이션 델타 표 (downstream 필수 — gsts-sandbox / 팀git / mwe)
| 우리 옛 API (group A, 폐기됨) | upstream 등가 API (채택) |
|------------------------------|--------------------------|
| `g.server().on(...).onSignal<Args extends readonly SignalArgDef[]>(name, handler)` | `g.server().on(...).onSignal<S extends SignalDefinition>(signalDef, (evt, f) => …)` — `evt.params.<paramName>` 타입드 접근 |
| 시그널 정의 = `onSignal`의 제네릭 `Args`(SignalArgDef[]) 인라인 | `defineSignal('<name>', [['<param>','<type>'], …]) → SignalDefinition` (별도 정의 객체, `Signal.x`로 재사용) |
| `f.sendSignal(target, name, ...signalArgs)` (signalArgs = 우리 인코딩) | `f.sendSignal<S extends SignalDefinition>(signalDef, ...params: SignalParamValues<S>)` 타입드 / `f.sendSignal(nameStr, ...params)` 언타입드. (nodes.ts:7409-7411) |
| `SIGNAL_ARG_TYPE_MAP`(18종 typeId/dataGroup/elementType, mappings.ts) | upstream 핀 인코딩 = `send_signal:300000`/`monitor_signal:300001` 노드 + per-param InParam/OutParam 핀(index.ts). 타입맵 직접 노출 없음 — `SignalParamType`(19종, faction 포함)로 표현 |
| 핸들러 인자 접근 = (우리 방식 — 인덱스/asType 추정) | `evt.params.<한글/임의 paramName>` (정의 순서·이름 기반 타입드 객체) |
| 시그널 추출 = 우리 `gsts inspect`(노드그래프 일반) | upstream `gsts inject` 자동추출 → `src/resources/signals.ts`(`extractSignalsFromGil`). 우리 inspect/scaffold는 별도 보존(노드그래프 검사·스캐폴딩 용도). |

> downstream 마이그레이션: `onSignal<Args>` 사용처 → `defineSignal` + `onSignal(def)` + `evt.params.x`로 변경. `sendSignal(...signalArgs)` → `sendSignal(def, ...params)`. (verdict-017이 leader에 전달.)

---

## 워킹트리 위생
- temp worktree(upstream 회귀확인용) 제거 + prune 완료(`git worktree list` = 메인만).
- scratch(scaffold smoke 출력 temp dir, bash.exe.stackdump, test-gen 잔여 enum .ts) 제거. tests/·dist 생성물 git checkout 복원.
- 최종 워킹트리(non-.claude): B/C src 변경 + `package-lock.json`(npm install 결과, 정당) + `.vs/`(세션 시작전부터 존재한 에디터캐시). stray 없음.

## self-eval 요약
| G | 판정 | 근거 |
|---|------|------|
| G0 | **met** | NEW_BASE=9cb31c8≠71ca7a1, 게이트 통과 |
| G1 | **met** | 백업태그·integ-upstream·master(8629dc9) 불변 확인 |
| G2 | **met** | A 전폐기·upstream 채택·B/C 재이식, diff=B/C만. proto upstream 채택. (B1 위치분산 주석) |
| G3 | **met** | B1=9(분산:binary7+signal_nodes2)/B2=1/B5=1. §4 롤백 없음 |
| G4 | **met** | install/tsc/build/scaffold-tsc/inspect/scaffold = EXIT0. signal e2e 핵심단계 PASS. npm test EXIT1은 upstream 선존버그(worktree 입증)·우리무관 |
| G5 | **방향B 실행 / superset 관찰 (폐기확정 review 대기)** | 폐기 group A 명시 + upstream 등가 매핑. 동등성 갭 없음(upstream⊇우리+faction). 마이그레이션델타표 작성. 폐기 확정은 review 독립검증 |

### review에 위임 (Org 제약)
1. G3/G4 grep·검증 **독립 재실행**(impl 로그 신뢰 금지) — 특히 B1 분산(binary.ts+signal_nodes.ts 합산) 확인.
2. G2 충돌해소 정합성 — A 코드 부재 + B/C 정확 재이식.
3. **G5 동등성 독립확인** — upstream defineSignal의 18종+_list 자동래핑이 우리 A와 동등함을 부록 기준으로 판정. fixture `src/resources/signals.ts` 확보해 assert-signal-parameters 300000/300001 핀 깊이검사 완주 권장.
4. npm test `other.literal.ts` 실패가 upstream 선존인지 독립 재현(worktree).
