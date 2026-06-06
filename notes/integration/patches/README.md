# Patch Bundle — signal-args HIGH 패치셋 + 종속 변경 (재적용 단위)

> Directive #014 First work item / 산출물 계약(contract-deliverables.md) 준수.
> 분기점(baseline): upstream `71ca7a1` (선형). 권위 델타 = `git diff 71ca7a1 HEAD -- src/` (20 src 파일, +1037/-26).
> 형식 결정: 단일 선형 히스토리의 커밋 경계(`66f4803` feat + `9415773` fix + `594a6e8` scaffold codegen fix #015)가 논리 그룹 A~E와 일치하지 않으므로, `git format-patch`(커밋 단위) 대신 **분기점 기준 파일별 diff(`git diff 71ca7a1 HEAD -- <file>`)** 로 캡처. 계약이 명시적으로 허용한 "explicit hunk 문서 + 파일별 diff" 방식.
> Refresh 이력: #015(`594a6e8`)로 `gil_scaffold.ts` 누적 델타가 236→251줄로 변경 → `C2-cli-gil_scaffold.patch` 재생성(2026-06-07). 나머지 19패치는 무변(바이트 일치 확인).

---

## 재적용 방법 (Reapply)

분기점(또는 upstream 최신본) 위에서 워킹트리 루트에서:

```bash
# 개별 적용 (권장 — 충돌 파일만 격리)
git apply notes/integration/patches/A01-runtime-value.patch
# ... 또는 그룹/전체 일괄
git apply notes/integration/patches/*.patch
```

- 모든 패치는 `71ca7a1` 기준으로 **개별·일괄 모두 clean apply 검증 완료** (`git apply --check`, 2026-06-06, 임시 worktree).
- upstream이 동일 파일을 수정해 충돌나면 `git apply --3way` 후 워크플로 문서(`../integration-workflow.md`) §2 HIGH 충돌 해소 가이드 참조.

---

## 매핑 표 (패치 ↔ 변경 그룹 ↔ 인벤토리 항목 ↔ 위험도)

계약의 그룹 A~E 분할 기준. 인벤토리 항목번호는 `notes/fork-changes-inventory.md`.

### 그룹 A — signal-args core (runtime + definitions)

| 패치 파일 | 대상 파일 | 인벤토리 | 위험도 |
|-----------|-----------|----------|--------|
| `A01-runtime-value.patch` | `src/runtime/value.ts` | A1 (`SignalArgDef`, `SignalArgsToPayload` 타입) | MED |
| `A02-runtime-core.patch` | `src/runtime/core.ts` | A2 + **B3** (`onSignal<Args>` 제네릭, signalArgs 전파, output pin) | **HIGH** |
| `A03-runtime-ir_builder.patch` | `src/runtime/ir_builder.ts` | A3 (`buildSignalNode`, signalParams) | MED |
| `A04-runtime-meta_call_types.patch` | `src/runtime/meta_call_types.ts` | A4 (`signalParams` 필드) | LOW |
| `A05-runtime-IR.d.patch` | `src/runtime/IR.d.ts` | A5 (`Node.signalParams`) | LOW |
| `A06-runtime-server_globals.patch` | `src/runtime/server_globals.ts` | A6 (`send(args?)`) | MED |
| `A07-runtime-server_globals.d.patch` | `src/runtime/server_globals.d.ts` | A7 (시그니처 선언) | LOW |
| `A09-definitions-events-payload.patch` | `src/definitions/events-payload.ts` | A9 (`value` import) | LOW |
| `A8B5-definitions-nodes.patch` | `src/definitions/nodes.ts` | **A8** (sendSignal+assemblyList) **+ B5** (parseValue entity fallback) | **HIGH** |

> **A8B5 분할 주의:** `nodes.ts` 단일 파일에 A8(sendSignal, 그룹 A)과 B5(parseValue, 그룹 D)가 공존. 4개 hunk가 import 영역에서 인접·교차하므로 물리적 hunk 분리 시 양쪽 패치가 독립 clean apply 불가 → **파일 단위 단일 패치로 유지하고 두 그룹에 동시 매핑**. hunk 내역은 아래 "A8B5 hunk 명세" 참조.

### 그룹 B — compiler (ir_to_gia_transform)

| 패치 파일 | 대상 파일 | 인벤토리 | 위험도 |
|-----------|-----------|----------|--------|
| `A10-compiler-mappings.patch` | `.../mappings.ts` | A10 (`SIGNAL_ARG_TYPE_MAP` 18종) | **HIGH** |
| `A11-compiler-index.patch` | `.../index.ts` | A11 + **B4** (핀 생성, `idx-1` 매핑, monitor_signal output pin) | **HIGH** |
| `A12-compiler-pins.patch` | `.../pins.ts` | A12 (`ensureInputPinWithType`, entity null 핀) | **HIGH** |

### 그룹 C — injector signal (signal-args + 버그 B1)

| 패치 파일 | 대상 파일 | 인벤토리 | 위험도 |
|-----------|-----------|----------|--------|
| `A13B1-injector-signal_nodes.patch` | `src/injector/signal_nodes.ts` | **A13** (ClientExec kind=5 우선탐색) **+ B1** (varint overflow guard 3함수) | **HIGH** |

### 그룹 D — bug fixes (독립)

| 패치 파일 | 대상 파일 | 인벤토리 | 위험도 |
|-----------|-----------|----------|--------|
| `B2-shared-ts_type_utils.patch` | `src/shared/ts_type_utils.ts` | **B2** (`any` 타입가드 1줄) | MED |
| *(B5는 위 `A8B5-definitions-nodes.patch`에 포함)* | `src/definitions/nodes.ts` parseValue | **B5** (entityLiteral bigint/int 폴백) | MED(파일은 HIGH) |

### 그룹 E — CLI / Reader / i18n (충돌 LOW, HIGH와 분리)

| 패치 파일 | 대상 파일 | 인벤토리 | 위험도 |
|-----------|-----------|----------|--------|
| `C1-cli-gil_inspect.patch` | `src/cli/gil_inspect.ts` (신규) | C1 | LOW |
| `C2-cli-gil_scaffold.patch` | `src/cli/gil_scaffold.ts` (신규) | C2 | LOW |
| `C3-injector-reader.patch` | `src/injector/reader.ts` (신규) | C3 | LOW |
| `C4-cli-gsts.patch` | `src/cli/gsts.ts` | C4 (inspect/scaffold 등록) | MED |
| `C5-i18n-en-US.patch` | `src/i18n/locales/en-US/main.json` | C5 | LOW |
| `C6-i18n-zh-CN.patch` | `src/i18n/locales/zh-CN/main.json` | C6 | LOW |

> **D1 (.gitignore):** 코드 영향 0 + `.claude/` 관리 대상 외 + src 미해당 → 패치 번들 비포함(의도적). 인벤토리 D1은 설정 변경으로 워크플로 추적표에만 기록. (E1~E4 문서 자산도 동일하게 코드 영향 0 → 비포함.)

---

## A8B5 hunk 명세 (`A8B5-definitions-nodes.patch`)

단일 파일에 두 인벤토리 항목이 4 hunk로 공존:

| hunk | 위치(분기점 기준) | 내용 | 매핑 |
|------|-------------------|------|------|
| 1 | `@@ -27` import | `+ entityLiteral` | B5 의존 |
| 2 | `@@ -47` import | `+ SignalArgDef` | A8 의존 |
| 3 | `@@ -237` `parseValue` entity case | `+ z.union([z.int(),z.bigint()])` → `new entityLiteral(...)` 폴백 | **B5** |
| 4 | `@@ -6807` `sendSignal` | `signalArgs?` 매개변수 + `_list` assemblyList 자동래핑 + `signalParams` | **A8** |

재적용 시 두 그룹(A, D)이 이 한 파일을 공유함을 인지. 충돌 발생 시 hunk 3(B5)은 **절대 드롭 금지**(워크플로 버그체크리스트 참조).

---

## 재적용 순서 / 종속성

레이어 의존 순서(아래→위). 같은 그룹 내 순서 무관, 그룹 간은 권장 순서:

1. **A (core 타입·런타임)** 먼저 — `value.ts`(A1)가 `SignalArgDef`를 export → `nodes.ts`(A8)·`core.ts`(A2)가 import. `value.ts` 선적용 권장.
2. **B (compiler)** — A의 IR 필드(`signalParams`)·타입에 의존.
3. **C (injector signal)** — 런타임/컴파일러 산출 GIA 구조에 의존(A13). B1 가드는 독립.
4. **D (독립 버그)** — B2는 완전 독립(언제든). B5는 A8B5 파일에 포함되어 A와 함께 적용됨.
5. **E (CLI/Reader/i18n)** — 신규 파일 위주, 다른 그룹 무관. 마지막 또는 병렬.

> 실무상 `git apply notes/integration/patches/*.patch` 일괄 적용이 검증됨(분기점 clean). 충돌은 HIGH 7파일에서만 발생 가능 → 워크플로 §2 참조.

---

## 완전성 체크 (코드 영향 항목 ↔ 패치 커버리지)

| 인벤토리 | 캡처 위치 |
|----------|-----------|
| A1~A13 | A01~A13 패치 (A8은 A8B5에) ✔ |
| B1 | A13B1 (signal_nodes) ✔ |
| B2 | B2 패치 ✔ |
| B3 | A02 (core.ts, A2와 동일 hunk) ✔ |
| B4 | A11 (index.ts, A11과 동일 hunk) ✔ |
| B5 | A8B5 (nodes.ts hunk 3) ✔ |
| C1~C6 | C1~C6 패치 ✔ |
| D1 | 비포함(설정, 추적표 기록) — 의도적 |

코드 영향 22항목(A1~A13, B1~B5, C1~C6) 전부 캡처. D1은 비코드 설정으로 추적표 전용.
