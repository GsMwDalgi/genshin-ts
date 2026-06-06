# Task Spec — fork-sync~impl — Directive #017 Upstream Merge (실행) — REV2 (방향 B 확정)

> 실행 노드 = `fork-sync~impl` (영속). result → `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-017.md`.
> 계약: `.claude/tree/gsts/nodes/fork-sync/workspace/contract-017-upstream-merge.md` (먼저 정독 — 경로/규약/G0~G5 수용기준).
> 권위 절차: `notes/integration-workflow.md` §1~§5 + 부록. 분류: `notes/fork-changes-inventory.md`. 번들: `notes/integration/patches/`.
> **이전 escalation 결과 반영**: 네 STEP1 rebase 발견(upstream v0.1.10이 signal-args 독립구현) → leader+user **방향 B 확정**. 이 REV2가 그 결정을 반영한 갱신본이다. STEP2가 "재구현"→"폐기·채택"으로 바뀜. STEP0/STEP1 사전작업(백업태그·브랜치·upstream remote)은 이미 완료됨 — 재확인만.

## ★ 방향 결정 = B (discard A + adopt upstream defineSignal) — leader+user 확정, autonomous
- directive `017-upstream-merge.md` "## Leader Decision" 정독.
- **우리 group A(A1~A13 signal-args) 폐기, upstream `defineSignal()`/typed `evt.params` 채택.** (미션: upstream 동일기능 도입 시 변형 버리고 채택 = 변형 최소화.)
- **단, 폐기 확정 전제 = review 독립 동등성 확인** (네가 아니라 review가 18종 타입 커버리지 + `_list` 자동래핑 동등성 검증). 너는 폐기·채택을 **실행**하되, 동등성 갭이 보이면 result에 명기 → review가 확정 → 갭은 C-하이브리드로만 보강(우리 기능 손실 금지).
- **모든 안 공통 보존(불변)**: B1/B2/B5 (upstream 미보유 → upstream 새 구조 위 재이식, §4 절대 롤백금지). C CLI(inspect/scaffold — gsts.ts 등록충돌만 해소). proto/gia_gen/thirdparty = upstream 그대로.
- **마이그레이션 델타 문서화 필수**(verdict가 leader에 전달): 우리 API(`sendSignal(...signalArgs)` / `onSignal<Args>` / `SIGNAL_ARG_TYPE_MAP`) → upstream 등가 API 매핑을 result에 표로 기록(downstream: gsts-sandbox/팀 git/mwe가 따라야 함).

## Goal
upstream `josStorer/genshin-ts` v0.1.10(`9cb31c8`)을 우리 fork에 통합. **signal-args는 upstream 네이티브 채택(방향 B)**, B1/B2/B5 버그수정·C CLI 도구는 보존. 3축 미션: upstream 변형 최소화(2축), 통합 상태 지속(3축), 우리 고유 변형(B/C) 유지(1축 — A는 upstream 동등도입으로 예외 폐기).

## Mode = autonomous
직접 판단·실행. 단 아래 가드는 절대 위반 금지. 막히면 STATUS_REPORT로 fork-sync(parent)에 escalate.

## 절차 (순서 엄수)

### STEP 0 — 사전 재확인 (이미 완료 — 검증만, G0/G1)
이전 세션에서 완료됨(merge-log-017.md 참조). 새로 다시 만들지 말고 **존재 확인만**:
- 백업 태그 `fork-backup-20260607` == `8629dc9` (`git rev-parse fork-backup-20260607`). 없으면 생성.
- `upstream` remote 등록·fetch 완료 (`git rev-parse upstream/master` == `9cb31c8` = v0.1.10 = NEW_BASE).
- `master` == `8629dc9` 불변. **이후로도 master force·이동 절대 금지.**
- G0 = NEW_BASE 9cb31c8 != 71ca7a1 → 머지 진행(게이트 통과 확정).
- 워킹트리 src code-clean (`git status --short`). 이전 rebase abort로 무오염 상태여야 함.

### STEP 1 — 작업 브랜치 (G1)
- `integ-upstream` 브랜치 사용(이미 존재 == master). 여기서 작업. **master force 금지.**
- 우리 커밋 레인지(참고, merge-log 확정값): `git log --oneline 71ca7a1..HEAD` = 9 커밋 (`eda452b, 66f4803, c16dfa7, d2320e8, 9415773, a216ac9, 5da1cb4, 594a6e8, 8629dc9`). 66f4803=대형 feature(여기에 group A 포함).
- **방향 B라 단순 rebase 재구현이 아님**. 권장 접근(택1, 네 판단):
  - **(권장) 베이스 교체 + 선별 재적용**: `integ-upstream`를 `upstream/master`(9cb31c8)로 reset/재생성 → 우리 것 중 **유지 대상만** 그 위에 재적용:
    - 유지: B1(varint guard, upstream signal_nodes 새 구조 위 재이식), B2(any-guard), B5(entityLiteral parseValue), C(gil_inspect.ts/gil_scaffold.ts/reader.ts 신규파일 + gsts.ts 등록), E(notes/.claude 문서 — 코드무관), proto는 upstream것 자동.
    - **폐기**: group A 전체(A1~A13) — 재적용 안 함. upstream defineSignal이 대체.
    - 번들 패치 활용 가능: `notes/integration/patches/`에서 B/C 계열만 골라 `git apply --3way`. A 계열(A01~A13, A8B5의 A8 부분)은 **적용 안 함**(단 A8B5의 B5 hunk와 A13B1의 B1 hunk는 버그수정이라 그 부분만 추출 재적용).
  - (대안) rebase 후 A hunk만 폐기: 충돌마다 upstream판 채택(A 폐기) + B/C hunk만 우리 것. A8B5/A13B1처럼 A와 B가 같은 패치에 섞인 곳은 B 부분만 살림. 복잡하면 권장안으로.
- 어느 방식이든 결과 = upstream v0.1.10 + (B1/B2/B5 + C) 만, group A 없음.

### STEP 2 — 방향 B 적용: A 폐기 + upstream 채택 + B/C 재이식 (G2)
순서(레이어 의존): value.ts → core.ts → nodes.ts → mappings.ts → index.ts → pins.ts → signal_nodes.ts, 그리고 B2(ts_type_utils.ts), C(cli/*, reader.ts, gsts.ts).

**원칙**: 각 HIGH-7 파일에서 **upstream 버전을 베이스로** 두고, 우리 것 중 **버그수정(B)·고유도구(C)만** 그 위에 얹는다. group A 코드는 재적용하지 않는다.

| 파일 | 처리 |
|------|------|
| `runtime/value.ts` | upstream 채택. 우리 A1(`SignalArgDef`/`SignalArgsToPayload`)은 폐기(upstream `SignalParamType` 등이 대체). 다른 곳이 이 심볼 참조하면 폐기로 제거됨 확인. |
| `runtime/core.ts` | upstream `onSignal<S extends SignalDefinition>`/`defineSignal` 채택. 우리 A2(`onSignal<Args>`)·B3 폐기. |
| `definitions/nodes.ts` | upstream sendSignal 채택(A8 폐기). **단 B5(parseValue entity → `new entityLiteral` + import)는 upstream에 없음 → 재이식 필수**(§4). upstream nodes.ts parseValue 위치에 B5 hunk 얹기. |
| `mappings.ts` | upstream `send_signal:300000`/`monitor_signal:300001` 채택. 우리 A10 `SIGNAL_ARG_TYPE_MAP` 폐기. |
| `index.ts` | upstream 핀생성 루프 채택. 우리 A11·B4 폐기. |
| `pins.ts` | upstream 채택. 우리 A12(`ensureInputPinWithType`) 폐기(upstream이 동등 핀생성). |
| `injector/signal_nodes.ts` | upstream 채택(A13 폐기). **B1 varint guard(3함수 offset/len/dataEnd/newOffset 가드)는 upstream 미보유 → upstream 새 구조 위 재이식 필수**(§4). upstream의 readField*/parseNodeGraphId 함수에 가드 삽입. |
| `shared/ts_type_utils.ts` (B2) | **B2 any-guard 보존**(`if (type.flags & ts.TypeFlags.Any) return false`). A 폐기로 효용이 줄 수 있으나 일반 안전장치라 유지(§4). 단 upstream이 동등처리 도입했으면 result에 명기. |
| C: `cli/gil_inspect.ts`,`cli/gil_scaffold.ts`,`injector/reader.ts` | 신규파일 — 그대로 추가. upstream 미보유. |
| C: `cli/gsts.ts` | upstream판 채택 + 우리 inspect/scaffold 서브커맨드 등록 재이식(upstream도 gsts.ts 수정 → 두 등록 공존). upstream `gil_signals.ts`는 다른 파일이라 충돌 아님. |
| proto/gia_gen/thirdparty | upstream 그대로(STEP3). |

- **버그수정 추출 주의**: A8B5 패치는 A8(폐기)+B5(보존)가 섞임 → **B5 hunk만** 적용. A13B1 패치는 A13(폐기)+B1(보존) → **B1 hunk만** 적용. 패치를 통째로 적용하면 A가 딸려옴 — hunk 선별하거나 upstream 베이스에 수동 재이식.
- **동등성 갭 관찰**: 채택한 upstream defineSignal이 우리 A의 (1)18종 타입 (2)`_list` 자동래핑을 못 덮는 부분이 보이면 result G5에 명기(폐기 확정은 review). 갭 확정 시에만 C-하이브리드 보강.
- 각 파일 처리 결과를 result-017.md G2 표에 기재.

### STEP 3 — proto 채택 (G2 보조)
- `.proto`/`gia_gen`/`thirdparty` 충돌은 우리쪽 버리고 upstream 그대로(우리 변경 0). 충돌 마커 보이면 upstream 채택.

### STEP 4 — B1/B2/B5 무결성 (G3, §4 절대 롤백금지)
머지/충돌해소 후 grep으로 확인(하나라도 없으면 머지 미완료):
- B1: `grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/signal_nodes.ts` → 8건 이상.
- B2: `grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts` → 1건.
- B5: `grep -n "new entityLiteral" src/definitions/nodes.ts` → 1건 이상.
- 카운트를 result-017.md에 기재.

### STEP 5 — 검증 (G4, 구체 증거 + EXIT 코드)
1. `npm install` (upstream package.json 변경 가능). 결과 기재.
2. typecheck: `npx tsc --noEmit` — 단 package.json scripts에 typecheck/lint 정의 있으면 그걸 우선. **EXIT 코드 기재**(0 필수).
3. build: `npm run build`(또는 정의 스크립트). 결과/EXIT 기재.
4. §4 grep 3종(STEP4와 동일) 재확인.
5. injection smoke:
   - signal-args end-to-end: **upstream `defineSignal`/typed `evt.params` API**로 시그널 인자 샘플 컴파일 → GIA 핀 생성됨 확인(우리 옛 `sendSignal(...signalArgs)` 아님 — A 폐기됨). upstream 테스트/예제 있으면 활용. (이게 동등성 실증 — review가 독립 재확인.)
   - CLI 회귀: `gsts inspect <샘플.gil>` 및 `gsts scaffold <샘플.gil> --id <id>` (샘플 = `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil`, id 예: AstroGear_05_Aegis 1073741920). scaffold 산출 TS는 tsc 통과까지 확인(#015 회귀와 동일 성격). 출력은 R: 또는 temp로 — 루트 오염 금지.

### 워킹트리 위생
- 모든 scratch(빌드 검증/scaffold 출력)는 temp worktree 또는 R: 램디스크. repo 루트에 stray 파일 남기지 말 것. 끝나면 `git status --short`로 작업브랜치 상태 깔끔히.

## result-017.md 작성 (G0~G5 self-eval, 파일 끝)
- G0: NEW_BASE 해시(9cb31c8), 게이트 판정(머지 진행 확정).
- G1: 백업태그명(fork-backup-20260607), 작업브랜치명(integ-upstream), master 불변 확인(`git rev-parse master`==8629dc9).
- G2: HIGH-7 표(파일별 처리: upstream채택+A폐기 / B·C 재이식). proto 채택.
- G3: B1/B2/B5 grep 카운트(upstream 베이스 위 재이식 후에도 8/1/1 이상).
- G4: install/tsc(EXIT=0)/build/smoke 증거. signal-args smoke는 **upstream API**로.
- **G5 (방향 B 핵심)**: 폐기한 group A 명시(A1~A13 + 대응 upstream 등가). 부록 동등성 관찰(18종/`_list` 자동래핑 — upstream이 덮는지, 갭 있으면 명기). **폐기 확정은 review** → 너는 "방향 B 실행함 + 관찰한 동등성/갭" 보고. **잠정**이 아니라 "leader 확정 방향 B 실행, review 동등성 게이트 대기"로 표기.
- **★ 마이그레이션 델타 표(필수)**: 우리 옛 API → upstream 등가. 최소:
  - `sendSignal(target, name, ...signalArgs)` → upstream `?`
  - `onSignal<Args extends SignalArgDef[]>` → upstream `onSignal<S extends SignalDefinition>` / `defineSignal()`
  - `SIGNAL_ARG_TYPE_MAP`(18종) → upstream `send_signal:300000` 등 핀 인코딩
  - (downstream gsts-sandbox/팀git/mwe가 이 표로 마이그레이션.)
- 각 G에 met/partial/unmet + 근거. **master는 force 안 함**(review PASS + §5 + leader 수용 후 별도 단계).

## escalate 트리거 (parent=fork-sync로 STATUS_REPORT)
- (signal-args 방향은 **이미 B로 확정** — 더 이상 escalate 불요. 그대로 실행.)
- B/C 재이식이 upstream 새 구조와 근본 비호환일 때(B1을 upstream signal_nodes에 못 얹는 등 구조 대격변).
- §5 검증이 upstream 변경 자체로 실패하고 우리 변경과 무관할 때.
- upstream defineSignal이 우리 A의 핵심(_list 자동래핑 등)을 **전혀** 못 덮어 C-하이브리드 보강 범위가 사실상 A 재구현 수준일 때(방향 재검토 신호).

## 자가검수 금지
구현만. 독립 검수는 fork-sync~review가 별도 수행.
