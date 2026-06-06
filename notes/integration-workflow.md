# 통합 워크플로 (Integration Workflow) — upstream 머지 절차

> Directive #014 First work item / 계약(contract-deliverables.md §통합 워크플로 형식) 6항목 충족.
> 적용 시점: upstream(josStorer/genshin-ts) repo URL 확보 후 실제 머지 시.
> 현재 분기점: `71ca7a1 update proto for client nodes` (선형). 권위 델타 = `git diff 71ca7a1 HEAD -- src/` (20 src 파일).
> 패치 번들: `notes/integration/patches/` (분기점 clean apply 검증 완료). 매핑/순서는 그 README.

---

## 1. 머지 전략 + 분기점 갱신 절차

**전략: rebase-on-upstream (권장)** — merge commit 대비.

| 비교 | rebase-on-upstream | merge upstream→fork |
|------|--------------------|--------------------|
| 히스토리 | 선형 유지(현재 포크가 이미 선형) | merge commit 생성 |
| 변형 가시성 | 우리 변경이 tip에 모임 → 재검토 쉬움 | 섞임 |
| upstream 추종 | "upstream 변형 최소화"(미션 3축) 부합 | diff 노이즈 |
| 충돌 | HIGH 7파일에 집중, 단계적 해소 | 한 번에 폭증 |

권장 이유: 포크가 이미 `71ca7a1` 위 선형이라 rebase가 자연스럽고, 우리 변형을 upstream tip 위에 깨끗이 재적재해 차이를 최소화·가시화한다(미션 3축).

### 갱신 절차

```bash
# 0. 사전: 워킹트리 clean, 현재 HEAD 백업 태그
git tag fork-backup-$(date +%Y%m%d) HEAD

# 1. upstream 등록 (URL 확보 후 1회)
git remote add upstream <UPSTREAM_URL>
git fetch upstream

# 2. 새 분기점 확인 — upstream tip 커밋 해시 기록
git log --oneline upstream/main -5     # NEW_BASE = upstream tip

# 3-A. (권장) rebase: 우리 커밋(66f4803, 9415773)을 NEW_BASE 위로
git rebase --onto upstream/main 71ca7a1 master
#   → 충돌 시 §2 가이드. 해소 후 git rebase --continue
# 3-B. (대안) 패치 재적용: NEW_BASE 체크아웃 후 번들 적용
#   git checkout -b integ upstream/main
#   git apply notes/integration/patches/*.patch   # 충돌 시 --3way

# 4. 분기점 갱신 기록: 이 문서·인벤토리·README의 "분기점 71ca7a1"을 NEW_BASE로 갱신
#    git diff <NEW_BASE> HEAD -- src/ 를 새 권위 델타로 재캡처(번들 재생성)
```

> proto 관련: 3단계에서 `.proto`/`gia_gen`/`thirdparty` 충돌은 발생하지 않음(우리 미변경, §3). upstream 것을 그대로 받음.

---

## 2. HIGH 7지점 재적용 순서 + 충돌 해소 가이드

HIGH 충돌 핫스팟 7파일(인벤토리 충돌표). 레이어 의존 순서대로 해소.

| 순서 | 파일 | 패치 | 인벤토리 | 충돌 시 핵심 |
|------|------|------|----------|--------------|
| 1 | `runtime/value.ts` | A01 | A1(MED, 선행 의존) | `SignalArgDef`/`SignalArgsToPayload` export 보존 — 하위 전부 의존 |
| 2 | `runtime/core.ts` | A02 | A2+B3 | `onSignal<Args>` 제네릭·`signalArgs` 전파·인자별 output pin. upstream이 onSignal 시그니처 바꿨으면 제네릭 매개변수 재부착 |
| 3 | `definitions/nodes.ts` | A8B5 | A8+B5 | sendSignal `signalArgs`+assemblyList(A8) **그리고** parseValue entity 폴백(B5). **두 hunk 모두 보존**(§4 체크리스트) |
| 4 | `compiler/.../mappings.ts` | A10 | A10 | `SIGNAL_ARG_TYPE_MAP` 18종 typeId/dataGroup/elementType. upstream이 타입맵 구조 바꿨으면 18종 항목을 신구조에 재매핑 |
| 5 | `compiler/.../index.ts` | A11 | A11+B4 | send_signal 입력핀/monitor_signal 출력핀 생성 + `idx-1` 매핑. 핀 생성 루프 위치 재배치 |
| 6 | `compiler/.../pins.ts` | A12 | A12 | `ensureInputPinWithType()` + entity null 핀 타입 처리 |
| 7 | `injector/signal_nodes.ts` | A13B1 | A13+B1 | ClientExec(kind=5) 우선탐색(A13) + **varint guard(B1)**. signal_nodes는 upstream 잦은 수정 핫스팟 → 충돌 확률 최고. B1 가드 절대 유지(§4) |

### 충돌 해소 일반 절차

1. `git apply --3way <patch>` 또는 rebase 충돌 마커 직면.
2. **우리 의도 우선 확인**: 해당 파일의 패치 + 인벤토리 항목 설명 + (필요시) `notes/changelog.md` #8 대조.
3. upstream 변경과 우리 변경이 **같은 함수**를 건드렸으면 → 우리 기능(signal-args)을 upstream 새 구조 위에 재구현. **삭제·우회 금지**.
4. upstream이 **동일 기능(signal-args)을 자체 도입**했으면 → 우리 구현 폐기·upstream 채택 검토(미션 1축 예외). 단 B1/B2/B5 버그수정은 upstream에 있는지 별도 확인(§4).
5. 해소 후 §5 검증.

---

## 3. proto 채택 (upstream 그대로)

- 우리 포크는 `.proto`/`gia_gen`/`thirdparty` **변경 0건**(인벤토리 §F, `git diff 71ca7a1 HEAD --stat`에 부재 확인).
- 따라서 머지 시 **upstream proto/gia_gen/thirdparty를 무조건 그대로 채택**. 충돌 없음.
- ⚠️ 주의: handoff/notes의 "신규 노드타입·필드명·Dictionary·ClientSignal(kind=6)" 서술은 **역분석 문서**이지 우리 코드 변경이 아님(인벤토리 §F). proto 관련 충돌이 보이면 우리 쪽을 버리고 upstream을 받는다.

---

## 4. 버그 수정 무결성 체크리스트 (B1/B2/B5 — 절대 롤백 금지)

머지/충돌 해소 **후 반드시** 아래를 확인. 하나라도 사라졌으면 머지 미완료.

- [ ] **B1 — varint overflow guard** (`injector/signal_nodes.ts`, 패치 A13B1)
  - 3함수(`readFieldMessages`/`readFieldBytes`/`parseNodeGraphId`) 모두에 가드 존재:
    - while 루프 조건 `offset >= 0 && offset < buf.length`
    - `if (len < 0) break`
    - `if (dataEnd > buf.length || dataEnd < 0) break`
    - `parseNodeGraphId`에 `if (newOffset < 0) break`
  - 검증: `grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/signal_nodes.ts` → 8건 이상.
  - 의미: 32bit 정수 오버플로우 → 인젝션 무한루프 방지. 가드 제거 시 무한루프 재발.
- [ ] **B2 — any 타입가드** (`shared/ts_type_utils.ts`, 패치 B2)
  - `isEntityLikeType` 함수 **시작부**에 `if (type.flags & ts.TypeFlags.Any) return false`.
  - 검증: `grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts` → 1건.
  - 의미: `any`가 entity로 오추론되어 중간변수 시그널 인자 오추론되는 것 방지. 제거 시 A 기능 오작동.
- [ ] **B5 — parseValue entity fallback** (`definitions/nodes.ts`, 패치 A8B5 hunk 3)
  - parseValue의 `case 'entity'`에 `z.union([z.int(), z.bigint()])` 성공 시 `new entityLiteral(result.data)` 반환.
  - `entityLiteral` import 존재(hunk 1).
  - 검증: `grep -n "new entityLiteral" src/definitions/nodes.ts` → 1건 이상.
  - 의미: `setCustomVariable`에 entity 값 전달 시 `Invalid value type: entity` 방지. **A8B5 충돌 해소 시 sendSignal(A8)만 챙기고 이 hunk를 흘리기 쉬움 — 최우선 주의.**

> B3/B4는 A2/A11과 동일 hunk라 그 파일 재적용 시 자동 보존(별도 체크 불요, 단 A2/A11 충돌 해소 시 함께 확인).

---

## 5. 재적용 후 검증 (빌드 / 타입체크 / 인젝션 스모크)

순서대로. 하나라도 실패 시 머지 미완료.

```bash
# 1. 의존성 (upstream package.json 변경 가능)
npm install

# 2. 타입체크 — A1/A2/A8 타입 전파가 깨지지 않았는지 (signal-args 핵심)
npx tsc --noEmit           # 또는 프로젝트 정의 typecheck 스크립트

# 3. 빌드
npm run build              # 프로젝트 빌드 스크립트

# 4. 버그수정 가드 존재 정적 확인 (§4 grep 3종)
grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/signal_nodes.ts
grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts
grep -n "new entityLiteral" src/definitions/nodes.ts

# 5. 인젝션 스모크 — signal-args end-to-end
#    - signalArgs를 가진 sendSignal/onSignal 샘플 컴파일 → GIA 핀 생성 확인
#    - CLI: gsts inspect <샘플.gil>  /  gsts scaffold <샘플.gil> --id <id>  (C 기능 회귀)
#    - 인젝터로 변환 후 varint 파싱 무한루프 없음(B1) 확인
```

> 검증 스크립트명은 upstream package.json 갱신에 따라 달라질 수 있음 → 머지 시 `package.json` scripts 확인 후 위 명령 대응.

---

## 6. 추적 표 (각 변경 그룹의 통합 상태)

상태: **미적용 / 적용 / 검증완료**. 실제 머지 진행 시 이 표를 갱신.

| 그룹 | 인벤토리 | 패치 | 위험도 | 상태 | 비고 |
|------|----------|------|--------|------|------|
| A core | A1~A9(A8 공유) | A01~A09, A8B5 | A2/A8 HIGH, 그외 MED/LOW | 미적용 | value.ts 선행 |
| B compiler | A10~A12 | A10,A11,A12 | HIGH×3 | 미적용 | mappings 18종 핵심 |
| C injector signal | A13 + B1 | A13B1 | HIGH | 미적용 | 최충돌 핫스팟 |
| D bug (독립) | B2, B5 | B2, A8B5(hunk3) | MED | 미적용 | §4 롤백금지 |
| (bug in A/B) | B3, B4 | A02, A11 | HIGH | 미적용 | A2/A11에 내포 |
| E CLI/Reader/i18n | C1~C6 | C1~C6 | LOW(C4 MED) | 미적용 | 신규파일 위주 |
| Config | D1(.gitignore) | (번들 외) | LOW | 미적용 | 설정, 추적만 |
| proto | (미변경) | — | — | upstream채택 | §3 |

> 현 단계(upstream URL 미확보)에서 전 그룹 "미적용"이 정상. URL 확보 후 §1→§2→§4→§5 진행하며 상태 갱신.

---

## 부록 — 우리 변형 폐기 판단 기준 (미션 1축 예외)

upstream이 다음을 자체 도입했을 때만 우리 구현 폐기·채택 검토:

| 우리 변형 | upstream 동등 기능 도입 시 |
|-----------|---------------------------|
| signal-args (A 전체) | upstream signal 커스텀 인자 지원 → 채택 검토. 단 18종 타입 커버리지·_list 자동래핑 동등성 확인 |
| B1 varint guard | upstream이 동일 가드 보유 시 우리 것 불필요. 없으면 PR 역제안 가능 |
| B2 any guard | upstream isEntityLikeType이 any 처리 시 불필요 |
| B5 parseValue | upstream parseValue가 entity bigint 폴백 보유 시 불필요 |
| C CLI/Reader | upstream에 동등 inspect/scaffold 도입 시 채택 검토(우리 독자 도구라 가능성 낮음) |

폐기 결정은 반드시 review 노드 독립 검증 후(미션 Org 제약).
