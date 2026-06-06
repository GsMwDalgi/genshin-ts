# Upstream Merge Audit — Directive #017 (독립 검수, 방향 B)

> 검수 노드: `fork-sync~review` (영속). impl 결과(`fork-sync~impl/workspace/result-017.md`)를 **신뢰하지 않고 독립 재실행**.
> 검수 대상 상태: `integ-upstream` 브랜치 HEAD = `9cb31c8` (= upstream/master, release v0.1.10) + 워킹트리 B/C 변경 (uncommitted). master = `8629dc9` 불변.
> 방향 = B 확정 (leader+user): group A(A1~A13 signal-args) 폐기 + upstream `defineSignal`/typed `evt.params` 채택. 폐기 확정 게이트 = R5 (Org 제약: review 독립 동등성 검증 후).

---

## G0 — NEW_BASE 판정 (독립 확인) — **PASS**
- `git rev-parse upstream/master` = `9cb31c8` (impl claim 일치).
- `git rev-parse master` = `8629dc9`; `git rev-parse 71ca7a1` = 분기점.
- `git merge-base --is-ancestor 71ca7a1 upstream/master` → exit 0 (true). 71ca7a1은 진짜 ancestor, upstream 전진.
- NEW_BASE(9cb31c8) ≠ 71ca7a1 → 머지 진행 게이트 정당. R1~R6 적용.

---

## R1 — 가역성 (G1) — **PASS**
독립 `git rev-parse` 출력:
| 검사 | 기대 | 실측 | 판정 |
|------|------|------|------|
| `fork-backup-20260607` | 8629dc9 | `8629dc9c08d9cc017272c1ac6a6076c8274e8a7a` | PASS |
| `master` (force 안 됨) | 8629dc9 | `8629dc9c08d9cc017272c1ac6a6076c8274e8a7a` | PASS |
| `integ-upstream` HEAD | 9cb31c8 | `9cb31c87c0598d229047055b88f70f9ab2bb8600` | PASS |
| 작업 브랜치 존재 | integ-upstream | `git branch` 확인됨 | PASS |
- master == backup tag == 8629dc9 → master 불변(force 흔적 0). 모든 작업 가역.

---

## R2 — 방향 B 적용 정합성 (G2) — **PASS**
### A 폐기 확인 (우리 명명이 코드에 0건이어야 정상)
- `grep -rnE "signalArgs|SignalArgsToPayload|SignalArgDef|SIGNAL_ARG_TYPE_MAP" src/` → **0건** (exit 1, no match). 워킹트리 전체(untracked C 파일 포함) 스캔. → A 폐기 **완전**.
- 대조: 백업태그(`fork-backup-20260607:src/compiler/ir_to_gia_transform/mappings.ts`)에는 `SIGNAL_ARG_TYPE_MAP` 18종 존재 확인 → 폐기 전후 정합.

### upstream signal 채택 확인
- `core.ts:115` `export function defineSignal<...>` 존재.
- `core.ts:335` `onSignal<S extends SignalDefinition>` 존재.
- `mappings.ts:421-422` `send_signal: 300000` / `monitor_signal: 300001` upstream 인코딩 존재.
- `index.ts:324` send_signal/monitor_signal per-param 핀생성 루프 존재.

### diff vs upstream = B/C만
- `git diff --stat upstream/master -- src/` = **10 파일, 832 insertions / 6 deletions** (impl claim 정확히 일치).
  - 신규(C): gil_inspect.ts(+153), gil_scaffold.ts(+251), reader.ts(+367).
  - 수정(B/C): gsts.ts(+24), nodes.ts(+5), en-US(+10), zh-CN(+10), binary.ts(+10/-6), signal_nodes.ts(+6/-2), ts_type_utils.ts(+2).
- group A 코드 0, proto/gia_gen은 upstream 그대로(diff 0). → 방향 B 정합.

---

## R3 — B1/B2/B5 무결성 (G3/§4, 절대 롤백금지) — **PASS** (위치분산 검증됨)
독립 grep 재실행 + diff vs upstream으로 "가드가 실제 fork 추가물인지"까지 확인:

| 버그fix | 파일 | grep 카운트 | 함수별 가드 (diff 확인) | 판정 |
|---------|------|------------|------------------------|------|
| B1 | `signal_nodes.ts` | **2** | parseNodeGraphId: `offset>=0`(L36) + `newOffset<0`(L56) | PASS |
| B1(분산) | `binary.ts` | **7** | readFieldBytes(L44 offset>=0/L54 len<0/L57 dataEnd<0) + readFieldMessages(L85/L95/L98) + upstream parseMessage 기존(L194) | PASS |
| B1 합계 | signal_nodes+binary | **9** | 3함수(readFieldBytes/readFieldMessages/parseNodeGraphId) 전부 가드 보존 | PASS |
| B2 | `ts_type_utils.ts` | **1** | isEntityLikeType 첫줄 `if (type.flags & ts.TypeFlags.Any) return false` (L28) | PASS |
| B5 | `nodes.ts` | **1** | parseValue case 'entity': `new entityLiteral(result.data)` (L250) + entityLiteral import (L35) | PASS |

- **위치분산 검증**: `git diff upstream/master -- src/injector/binary.ts`로 가드 3종(`offset>=0` while가드 / `len<0` break / `dataEnd<0` break)이 readFieldBytes·readFieldMessages 양 함수에 **fork가 추가한 라인**임을 확인(upstream 원본은 `while (offset < buf.length)` + `dataEnd > buf.length`만). signal_nodes.ts diff도 parseNodeGraphId에 `offset>=0` + `newOffset<0`이 fork 추가물임을 확인.
- impl이 경고한 "spec §4의 signal_nodes 8건 기대"는 upstream v0.1.10이 readField*를 binary.ts로 **이동**한 결과로 더 이상 단일파일 기준이 무효. 두 파일 합산 9건 + 함수별 가드 의미 보존 = 롤백 없음. signal_nodes 단독 2건만 보고 "롤백" 오판 금지(impl 주석 정당, 독립 재확인 완료).

---

## R4 — §5 검증 독립 재실행 (G4) — **PASS** (npm test EXIT1 = upstream 선존버그 독립 재현)
작업 브랜치(integ-upstream)에서 직접 실행, EXIT 코드 직접 확인:

| 단계 | 명령 | EXIT | 결과 | 판정 |
|------|------|------|------|------|
| install | `npm install` | **0** | 0 vuln | PASS |
| typecheck | `npx tsc --noEmit` | **0** | 클린(B/C가 upstream v0.1.10에 정합 컴파일) | PASS |
| build | `npm run build` | **0** | dist 생성 + proto/d.ts 복사 OK | PASS |
| §4 grep | (R3) | — | B1=9(분산)/B2=1/B5=1 | PASS |
| smoke: inspect | `gsts inspect 1073741976.gil` (5.4MB) | **0** | 184+ node graphs 정상 나열, B1 무한루프 없음 | PASS |
| smoke: scaffold | `gsts scaffold ... --id 1073741920 --out <temp>` | **0** | AstroGear_05_Aegis.scaffold.ts 생성. **#015 픽스 보존 확인**: `import { g } from 'genshin-ts/runtime/core'`, `vec3([0,0,0])`, `list('entity', [])` | PASS |
| smoke: signal e2e | `npx tsx scripts/assert-signal-parameters.ts` | 1(부분) | 핵심 단계(L164-188) 전부 PASS, L190 fixture 게이트서 중단 — 아래 상세 | 부분 PASS |
| full suite | `npm test` | 1 | `other.literal.ts:11:9 cannot infer list type` — 아래 독립 재현 | PASS(우리무관 입증) |

### ★ npm test EXIT=1 독립 재현 (중요) — 우리 머지 무관 입증
- 우리 트리: `npm test` → EXIT 1, `[error] cannot infer list type ... at other.literal.ts:11:9`.
- **PURE upstream worktree 독립 재현**: `git worktree add --detach D:/MyDrive/Repos/_gsts_upstream_check upstream/master` → `npm install`(EXIT 0) → `npm test` → **동일 EXIT 1, 동일 메시지** `cannot infer list type ... at other.literal.ts:11:9`.
- → upstream v0.1.10 **자체 선존버그**, 우리 B/C 머지와 무관. impl claim **독립 확인됨**. (검증 후 worktree remove + prune 완료.)

### ★ signal-args e2e 부분 PASS 상세
- `assert-signal-parameters.ts` 실행 → L190 `AssertionError: Expected extracted signals.ts to exist`에서 중단. 이는 **clean assert.ok 실패**로, 그 이전 L164-188 어서션이 **전부 통과**했음을 의미:
  - (a) 핵심 통과 확인: `defineSignal` emit, `f.sendSignal(Signal.signal_param_literal, int(...))` / wired / `f.sendSignal('signal_param_none')` 형태, typed `evt.params.参数_1/2/3` (L173-183) 통과.
  - (b) **옛 group-A 부재 입증**: `assertNotContains 'signalParam0.asType'`/`signalParam1`/`signalParam2`/`signalParam3` (L185-188) 통과 = 생성 GS에 우리 옛 signal 인코딩 흔적 0 → 방향 B 동작 + A 미잔존 재확인.
  - (c) L190 게이트 = 로컬 fixture `src/resources/signals.ts`(upstream 미커밋, `gsts inject`/`extractSignalsFromGil` 산출물) 부재. 스크립트·fixture를 우리가 미수정 → **우리 변경과 무관**(fixture 부재 게이트). 독립 확인됨.
- 300000/300001 핀 깊이검사(L214 이후)는 L190 fixture 게이트 뒤라 미도달. fixture 생성에는 inject config 설정 + src/ 쓰기(루트 오염) 필요 → 계약상 보류. **부분 PASS + 사유**(spec 허용 범위).

### scaffold tsc 주석
- 생성 scaffold는 `int()`/`vec3()`/`list()`/`prefabId()`/`bool()`/`float()`을 ambient 글로벌(server_globals.global.d.ts)로 사용(`g`만 import). 단독 tsc는 ambient 미주입으로 globals 미해소 — 이는 scaffold 출력의 **기존 ambient-globals 특성**(#015부터 동일)이지 머지 회귀 아님. 의미있는 스모크 = scaffold 생성 EXIT0 + #015 구문 보존(확인됨). impl의 "scaffold tsc EXIT0"은 dist 대상(ambient 주입)에서의 검증 — 패턴 정합.

---

## R5 — ★ group A 폐기 동등성 게이트 (G5/부록) — 이번 검수 중심 — **폐기 확정 PASS** (갭 없음)

### (1) 18종 타입 커버리지 — superset 독립 확인
- **우리 옛 A10 `SIGNAL_ARG_TYPE_MAP` 18종** (백업태그 `fork-backup-20260607:src/compiler/ir_to_gia_transform/mappings.ts` L406-423 직접 추출):
  - 스칼라 9: entity, guid, int, bool, float, str, vec3, config_id, prefab_id
  - 리스트 9: guid_list, int_list, bool_list, float_list, str_list, entity_list, vec3_list, config_id_list, prefab_id_list
- **upstream `SignalParamType`** (core.ts:48-69 직접 검사, 21항):
  bool, int, float, str, vec3, guid, entity, prefab_id, config_id, **faction**, bool_list, int_list, float_list, str_list, vec3_list, guid_list, entity_list, prefab_id_list, config_id_list, **faction_list**, unknown
- **집합 대조**:
  - ours − upstream = **∅** (우리 18종 전부 upstream에 존재) → **커버리지 갭 없음**.
  - upstream − ours = {faction, faction_list, unknown} → upstream이 진부분집합으로 더 넓음.
  - 결론: **upstream ⊇ ours (superset)**. impl claim 독립 확인됨.

### (2) `_list` 자동래핑 등가
- upstream `asSignalParamValue` (core.ts:147-191): 9종 `_list` 전부 `paramValue.asType('<x>_list')` 케이스 보유(L169-188). 즉 list 파라미터 타입드 처리 등가 제공.
- `index.ts` send_signal/monitor_signal 핀생성: `assembly_list` 노드 핸들러(L367) + multiple_branches의 `setLiteralArgValue(..., '${caseValueType}_list', ...)` (L396)로 list 값 타입 인코딩 경로 보유.
- e2e 어서션(assert-signal-parameters L202-211)이 float_list/str_list/vec3_list/bool_list/guid_list/entity_list/prefab_id_list/config_id_list/int_list 추출·정의를 검증(우리 트리에선 fixture 게이트 전 단계 통과로 정의/타입 경로는 동작 확인, 추출 산출 깊이검사는 fixture 게이트 뒤 미도달).
- 판정: 옛 A8 assemblyList 자동래핑의 등가(타입드 list 파라미터 처리 + 핀 인코딩 경로)가 upstream에 존재. **등가 확인**.

### R5 종합 판정 — **폐기 확정 PASS**
- 18종 커버리지 갭 없음(upstream superset) + `_list` 처리 등가 → 동등성 충족.
- **C-하이브리드 보강 불요**(갭 미관찰).
- Org 제약(미검증 폐기 = 치명 FAIL) 위반 없음 — 동등성 **독립 검증 완료** 후 폐기 확정.
- 비고: B1/B2/B5는 폐기 아님(R3에서 보존 확인). upstream가 동등 도입한 항목 아님(우리 고유 버그픽스, 유지).

---

## R6 — 마이그레이션 델타 정확성 — **PASS** (경미한 라인번호 드리프트 INFO)
impl 델타 표 독립 대조:
| 항목 | impl 표기 | 독립 확인 | 판정 |
|------|----------|-----------|------|
| `onSignal<Args>` → `onSignal<S extends SignalDefinition>` + `evt.params.x` | 맞음 | core.ts:335 확인 | PASS |
| 인라인 Args → `defineSignal('<name>', [['<param>','<type>'],…])` | 맞음 | core.ts:115 확인 | PASS |
| `sendSignal(...signalArgs)` → `sendSignal<S>(def, ...params)` 타입드 / `sendSignal(nameStr, ...params)` 언타입드 | 맞음 (impl "nodes.ts:7409-7411") | 실측 nodes.ts:**7414-7416** — 내용 정확, **라인번호만 5줄 드리프트** | PASS (INFO) |
| `SIGNAL_ARG_TYPE_MAP` → `SignalParamType`(faction 포함) + 300000/300001 핀 | 맞음 | mappings.ts:421-422 + core.ts:48-69 확인 | PASS |
| 옛 API 명(SignalArgDef/SIGNAL_ARG_TYPE_MAP/onSignal<Args>) | 폐기됨 | 현 트리 grep 0 + 백업태그 존재로 전후 정합 | PASS |
- INFO(경미, non-blocking): sendSignal 라인번호 표기 7409-7411 vs 실측 7414-7416. 내용 동일, downstream 마이그레이션 자료 정확성에 영향 없음(시그니처가 키, 라인번호는 참고).

---

## 워킹트리 위생 (검수 후)
- 검증용 임시 worktree(`D:/MyDrive/Repos/_gsts_upstream_check`) remove + prune 완료(`git worktree list` = 메인만).
- scaffold smoke 출력 temp(`D:/MyDrive/Repos/_gsts_scratch`) 제거. npm test가 재생성한 `tests/generated/`·`tests/enum_cases/` `git checkout`/제거로 복원.
- 최종 워킹트리(non-.claude): B/C src 변경(10파일) + `package-lock.json`(npm install 정당) + `.vs/`(세션전부터 존재). stray 0 — impl이 남긴 상태와 동일.

---

## 검수 항목 요약
| R | 항목 | 판정 | 분류 |
|---|------|------|------|
| G0 | NEW_BASE 판정 | **PASS** | — |
| R1 | 가역성 | **PASS** | — |
| R2 | 방향 B 정합성(A폐기·upstream채택·B/C재이식) | **PASS** | — |
| R3 | B1/B2/B5 무결성(위치분산) | **PASS** | — |
| R4 | §5 검증(install/tsc/build/smoke/test) | **PASS** | npm test EXIT1 = upstream 선존(독립재현). signal e2e 부분PASS(fixture 게이트, 우리무관) |
| R5 | ★ group A 폐기 동등성 게이트 | **폐기 확정 PASS** | 갭 없음(upstream⊇우리), _list 등가, C-하이브리드 불요 |
| R6 | 마이그레이션 델타 정확성 | **PASS** | INFO: sendSignal 라인번호 5줄 드리프트(경미) |

### 치명(blocking) 발견: **없음**
- §4 가드 누락 0, master force 0, 미검증 폐기 0.

### 경미(non-blocking) 발견
1. R4: signal e2e 300000/300001 핀 깊이검사 미완주 — fixture `src/resources/signals.ts` 부재(upstream 미커밋 + 우리 환경 미생성). 핵심 단계는 통과. 권장: leader 수용 단계에서 fixture 확보 시 깊이검사 완주(보강, 차단 아님).
2. R6: impl 델타 표 sendSignal 라인번호(7409-7411 → 실측 7414-7416). 내용 정확, 표기만 수정 권장.

---

## VERDICT — **PASS**
방향 B 머지(group A 폐기 + upstream defineSignal/typed evt.params 채택)는 독립 검수 전 항목 통과:
- A 폐기 완전(잔재 0), upstream signal 정상 채택, B1/B2/B5 무결(위치분산 9건 보존), tsc/build EXIT0, CLI 스모크(inspect/scaffold) EXIT0 + #015 픽스 보존.
- **R5 폐기 확정 게이트 충족**: 18종 커버리지 갭 없음(upstream superset), `_list` 등가 → Org 제약 만족(미검증 폐기 아님). C-하이브리드 보강 불요.
- npm test EXIT1은 upstream 선존버그(pure upstream worktree 독립 재현)로 우리 머지 무관.

차단 이슈 없음. 경미 2건은 비차단(권장 보강). fork-sync(parent)가 verdict-017 통합 → leader 수용 + master-update 진행 가능.
