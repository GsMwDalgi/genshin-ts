# Verdict — Directive #017: Upstream Merge Execution (방향 B)

> 통합 노드: `fork-sync` (distributor). 분할: impl=`fork-sync_impl`(실행), review=`fork-sync_review`(독립검수). impl≠review 준수.
> 결과 상태: 작업 브랜치 `integ-upstream` HEAD=`9cb31c8` (= upstream/master, v0.1.10) + 워킹트리 B/C 변경 (uncommitted). **master=`8629dc9` 불변.**
> **VERDICT = PASS.** 차단 이슈 0. master force·커밋은 leader 수용 후 별도 단계(내가 안 함).

---

## 1. 산출 요약 (user constraint 항목별)

| 항목 | 값 |
|------|------|
| **작업 브랜치** | `integ-upstream` (HEAD = `9cb31c8` + uncommitted B/C) |
| **백업 태그** | `fork-backup-20260607` = `8629dc9` |
| **NEW_BASE** | `9cb31c8` (upstream `josStorer/genshin-ts` release **v0.1.10**), 분기점 `71ca7a1`에서 +43 커밋. NEW_BASE ≠ 71ca7a1 → 머지 게이트 정당 |
| **master** | `8629dc9` 불변 (force 0) |
| **방향** | **B** (leader+user 확정): group A(A1~A13) 폐기 + upstream `defineSignal()`/typed `evt.params` 채택 |
| **src diff vs upstream/master** | 10 파일, **+832 / -6** — 전부 B/C. group A 코드 0 |
| **마이그레이션 영향** | downstream(gsts-sandbox/팀git/mwe)이 upstream signal API로 마이그레이션 (§6 델타표, leader 수용) |

---

## 2. HIGH-7 충돌해소 결과 (방식: reset --hard upstream/master + 유지대상 수동 재이식)

impl 방식 = `git reset --hard upstream/master`로 깨끗한 upstream v0.1.10 위에 **유지대상(B1/B2/B5 + C)만 재이식**, group A는 0줄 재적용. rebase 대신 채택한 이유: 첫 커밋(66f4803=group A)에서 HIGH-7 전수 충돌 → hunk별 A폐기가 오류유발적.

| HIGH-7 파일 | 처리 | A 폐기 | B/C 재이식 |
|-------------|------|--------|------------|
| `runtime/value.ts` | upstream 채택 | A1(SignalArgDef/SignalArgsToPayload) | — |
| `runtime/core.ts` | upstream `defineSignal`/`onSignal<S>` 채택 | A2 + 내포 B3 | — |
| `definitions/nodes.ts` | upstream sendSignal 채택 + **B5 재이식** | A8 | **B5** parseValue case 'entity' → `new entityLiteral(result.data)` (L250) |
| `compiler/.../mappings.ts` | upstream `send_signal:300000`/`monitor_signal:300001` | A10(18종 맵) | — |
| `compiler/.../index.ts` | upstream per-param 핀생성 루프 | A11 + 내포 B4 | — |
| `compiler/.../pins.ts` | upstream 채택 | A12 | — |
| `injector/signal_nodes.ts` | upstream 채택 + **B1 부분 재이식** | A13 | **B1** parseNodeGraphId 가드 (2건) |

추가 (HIGH-7 외, B1 위치이동·C):
- `injector/binary.ts`: upstream이 readFieldBytes/readFieldMessages를 signal_nodes→binary로 **이동** → **B1 주이식** (7건). → §4 주의 참조.
- `shared/ts_type_utils.ts`: **B2 재이식** (isEntityLikeType Any 가드).
- C 신규파일 (upstream 미보유, 충돌 0): gil_inspect.ts(+153), gil_scaffold.ts(+251, #015 픽스 포함), reader.ts(+367). `cli/gsts.ts`(+24, inspect/scaffold 서브커맨드 등록 — upstream gil_signals.ts와 다른 파일이라 코드충돌 없음). i18n 10키×2 로케일.
- proto / gia_gen / thirdparty: **upstream 그대로** (우리 변경 0, 충돌 0).

---

## 3. §4 — B1/B2/B5 무결성 체크리스트 (절대 롤백금지) — **PASS**

fork-sync **독립 git grep 재실행** (impl·review 로그 비신뢰):

| 버그fix | 위치 | grep 카운트 | 판정 |
|---------|------|------------|------|
| **B1** | `signal_nodes.ts` (parseNodeGraphId) | **2** | PASS |
| **B1** | `binary.ts` (readFieldBytes + readFieldMessages + upstream parseMessage 기존) | **7** | PASS |
| **B1 합계** | 두 파일, 3함수 전부 가드 보존 | **9** | PASS |
| **B2** | `ts_type_utils.ts:28` `TypeFlags.Any) return false` | **1** | PASS |
| **B5** | `nodes.ts:250` `new entityLiteral` | **1** | PASS |

**★ B1 위치분산 해석 (다음 세대 오판 방지)**: spec §4의 "signal_nodes.ts 8건 이상" 기대는 upstream v0.1.10이 readField* 함수를 binary.ts로 **이동**하기 전 기준. signal_nodes 단독 2건만 보면 롤백처럼 보이나, binary.ts 합산 9건으로 3함수(readFieldBytes/readFieldMessages/parseNodeGraphId)의 가드 의미(`offset>=0` while가드 / `len<0` break / `dataEnd<0` break / `newOffset<0` break)가 전부 보존. review가 `git diff upstream/master -- binary.ts`로 이 가드들이 fork 추가물(upstream 원본=`while(offset<buf.length)`+`dataEnd>buf.length`만)임을 독립 확인. **롤백 0.**

---

## 4. §5 — 검증 증거 (구체 EXIT) — **PASS**

review가 작업 브랜치에서 직접 재실행, EXIT 직접 확인:

| 단계 | 명령 | EXIT | 결과 |
|------|------|------|------|
| install | `npm install` | **0** | 0 vuln |
| typecheck | `npx tsc --noEmit` | **0** | 클린 (B/C가 upstream v0.1.10에 정합 컴파일) |
| build | `npm run build` | **0** | dist 생성 + proto/d.ts 복사 OK |
| §4 grep | (위 §3) | — | B1=9 / B2=1 / B5=1 |
| smoke: inspect | `gsts inspect 1073741976.gil` (5.4MB) | **0** | 184+ node graphs 정상, B1 무한루프 없음 |
| smoke: scaffold | `gsts scaffold ... --id 1073741920` | **0** | AstroGear_05_Aegis.scaffold.ts 생성. **#015 픽스 보존** (`import {g} from 'genshin-ts/runtime/core'`, `vec3([0,0,0])`, `list('entity', [])`) |
| smoke: signal e2e | `npx tsx scripts/assert-signal-parameters.ts` | 1 (부분) | 핵심단계(L164-188) 전부 PASS: defineSignal→GS 변환, typed `evt.params.参数_1/2/3`, sendSignal, **옛 group-A 부재**(`assertNotContains signalParam0.asType` 등 통과). L190 fixture 게이트서 중단 → §7 non-blocking-1 |
| full suite | `npm test` | 1 | `other.literal.ts:11 cannot infer list type` — **upstream 선존버그** (§5 참조) |

---

## 5. npm test EXIT=1 — upstream 선존버그 (우리 머지 무관, 독립 재현)

- 우리 트리: `npm test` → EXIT 1, `cannot infer list type ... at other.literal.ts:11:9`.
- review가 **pure upstream worktree 독립 재현**: `git worktree add --detach upstream/master` → npm install(EXIT 0) → npm test → **동일 EXIT 1, 동일 메시지**.
- → upstream v0.1.10 자체 선존버그. 우리 B/C 머지와 무관. (검증 후 worktree remove + prune.)

---

## 6. 동등성 게이트 (R5/부록) — group A 폐기 확정 — **PASS** (review 독립확인 + fork-sync 재대조)

Org 제약: 변형 폐기는 review 독립 동등성 검증 후만. review가 검증, **fork-sync가 집합대조를 디스크에서 직접 재확인**:

### (1) 18종 타입 커버리지 — superset 확정
- **옛 A10 `SIGNAL_ARG_TYPE_MAP` 18종** (backup tag `mappings.ts` L405-423 직접 추출): 스칼라 9 {entity, guid, int, bool, float, str, vec3, config_id, prefab_id} + 각 `_list` 9.
- **upstream `SignalParamType`** (core.ts L48-69 직접 검사, 21항): 우리 18종 전부 + {faction, faction_list, unknown}.
- **집합대조**: ours − upstream = **∅** (갭 없음). upstream − ours = {faction, faction_list, unknown}. → **upstream ⊇ ours (superset)**.

### (2) `_list` 자동래핑 등가
- upstream `asSignalParamValue` (core.ts L147-191) 9종 `_list` 전부 `asType('<x>_list')` 케이스 보유. index.ts 핀생성에 list 인코딩 경로 존재. 옛 A8 assemblyList 자동래핑 등가 확인.

### 판정
- 커버리지 갭 0 + `_list` 등가 → **동등성 충족, 폐기 확정 PASS**.
- **C-하이브리드 보강 불요** (갭 미관찰) → **impl 재활성화(C-patch) 불필요**.
- 미검증 폐기(치명 FAIL) 회피 — 동등성 독립검증 후 폐기. B1/B2/B5는 폐기 아님(우리 고유 버그픽스, 보존).

---

## 7. 마이그레이션 델타 (downstream 필수 — gsts-sandbox / 팀git / mwe; leader 수용)

| 우리 옛 API (group A, 폐기됨) | upstream 등가 API (채택) |
|------------------------------|--------------------------|
| `onSignal<Args extends readonly SignalArgDef[]>(name, handler)` | `onSignal<S extends SignalDefinition>(signalDef, (evt, f) => …)` — `evt.params.<paramName>` 타입드 접근 (core.ts:335) |
| 시그널 정의 = `onSignal` 제네릭 `Args` 인라인 | `defineSignal('<name>', [['<param>','<type>'], …]) → SignalDefinition` 별도 정의객체, `Signal.x` 재사용 (core.ts:115) |
| `f.sendSignal(target, name, ...signalArgs)` | `f.sendSignal<S>(signalDef, ...params: SignalParamValues<S>)` 타입드 / `f.sendSignal(nameStr, ...params)` 언타입드 (nodes.ts:7414-7416) |
| `SIGNAL_ARG_TYPE_MAP` (18종 typeId/dataGroup) | upstream 핀 인코딩 `send_signal:300000`/`monitor_signal:300001` 노드 + per-param 핀(index.ts). 타입 표현 = `SignalParamType` (faction 포함) |
| 핸들러 인자 = (옛: 인덱스/asType) | `evt.params.<paramName>` (정의 순서·이름 기반 타입드 객체) |
| 시그널 추출 = 우리 `gsts inspect` | upstream `gsts inject` 자동추출 → `src/resources/signals.ts`(`extractSignalsFromGil`). 우리 inspect/scaffold는 별도 보존(노드그래프 검사·스캐폴딩) |

> downstream 행동: `onSignal<Args>` → `defineSignal` + `onSignal(def)` + `evt.params.x`. `sendSignal(...signalArgs)` → `sendSignal(def, ...params)`.

---

## 8. fork-sync 독립 재확인 (사후, 디스크 직접)

impl·review 로그 비신뢰, fork-sync가 `git rev-parse`/`git grep`/`git show`/`git diff` 직접 재실행 — **전부 일치**:
- master = backup = `8629dc9` (force 0) / upstream/master = `9cb31c8` / HEAD = `integ-upstream` / `merge-base --is-ancestor 71ca7a1 upstream/master` exit 0.
- A 잔재: `git grep -nE "signalArgs|SignalArgsToPayload|SignalArgDef|SIGNAL_ARG_TYPE_MAP" -- src/` = **0건**.
- diff vs upstream/master = 10파일 +832/-6 (B/C만).
- B1 = signal_nodes 2 + binary 7 = 9 / B2 = 1 (ts_type_utils:28) / B5 = 1 (nodes:250).
- R5 집합대조 = 직접 재대조(§6): ours ⊆ upstream, 갭 ∅ → 폐기 확정 채택.
→ review VERDICT=PASS, R5 폐기 확정 PASS **채택**.

---

## 9. non-blocking (권장 보강, 차단 아님)

1. **signal e2e 300000/300001 핀 깊이검사 미완주** — fixture `src/resources/signals.ts`(upstream 미커밋 + 우리 환경 미생성) 부재로 L190 게이트서 중단. 핵심단계(defineSignal→GS, typed evt.params, 옛 A 부재)는 통과. fixture 생성에는 inject config + src/ 쓰기(루트 오염) 필요 → 계약상 보류. **권장**: leader 수용 단계에서 fixture 확보 시 핀 깊이검사 완주.
2. **R6 sendSignal 라인번호 드리프트** — impl 델타표 "nodes.ts:7409-7411" vs 실측 7414-7416. 내용·시그니처 정확, 라인번호 표기만(시그니처가 마이그레이션 키, 라인은 참고). 본 verdict §7은 실측값(7414-7416) 반영.

---

## 워킹트리 위생
임시 worktree(`_gsts_upstream_check`)·scratch(`_gsts_scratch`) remove + prune 완료. npm test 재생성물 복원. 최종 = B/C src 10파일 + package-lock.json(npm install 정당) + .vs/(세션전 존재). stray 0.

---

## leader 수용 안내 (master-update 시)
- **현재 master 불변** — 머지는 `integ-upstream`에만 존재. master force·커밋은 **leader 수용 후 별도 단계** (review PASS + §5 통과 충족됨).
- 수용 시: integ-upstream을 master로 (fast-forward 아님 — master=8629dc9는 9cb31c8의 ancestor 아니므로 force/reset 필요), 백업태그 `fork-backup-20260607`로 롤백 가능.
- 마이그레이션 스코프(§7)를 downstream에 surfacing 필요 — 시그널 API breaking change.
- 베이스라인 번들(`notes/integration/patches/`, README, audit) 갱신 필요: 분기점·델타가 9cb31c8 기준으로 바뀜 → 차기 fork-maintenance 작업(별 디렉티브)으로 처리 권장.

---

## SELF-EVAL (parent=leader 판단용)

**criteria 전부 met. VERDICT = PASS.**

- **branch+tag**: met — integ-upstream / fork-backup-20260607=8629dc9 (§1, fork-sync rev-parse 재확인).
- **NEW_BASE**: met — 9cb31c8 (v0.1.10), ≠71ca7a1, 게이트 정당 (§1, is-ancestor 재확인).
- **HIGH-7 outcome**: met — 방향 B 적용, A 전폐기 + B/C 재이식, diff=B/C만 +832/-6 (§2, §8).
- **§4 grep counts**: met — B1=9(분산)/B2=1/B5=1, 롤백 0, 위치분산 독립검증 (§3, fork-sync grep 재실행).
- **§5 evidence**: met — install/tsc/build/inspect/scaffold EXIT0, #015 픽스 보존. npm test EXIT1=upstream 선존(pure worktree 재현). signal e2e 부분 PASS(핵심 통과, fixture 게이트=우리무관) (§4, §5).
- **equivalence-gate (폐기 확정)**: met — R5 PASS. ours ⊆ upstream(18종 갭∅, superset) + _list 등가. review 독립검증 + **fork-sync 집합대조 디스크 직접 재확인** (§6). 미검증 폐기 회피. C-하이브리드 불요 → impl 재활성화 불필요.
- **migration delta**: met — §7 표 (실측 라인번호 반영). leader가 downstream에 전달.
- **impl≠review**: met — fork-sync_impl 실행, fork-sync_review 독립검수(grep/EXIT/git/pure-upstream-worktree 재현), fork-sync 통합·독립 재확인.

**partial**: §9-1 signal e2e 핀 깊이검사 — fixture 부재로 미완주(spec "불가 시 부분PASS+사유" 허용 범위). 핵심 e2e 입증됨 → 차단 아님.

**unmet**: 없음.

**blocked**: 없음.

> 차단 이슈 0. leader 수용 + master-update 진행 가능. **master force·커밋은 leader 소관 — fork-sync는 수행하지 않음.**
