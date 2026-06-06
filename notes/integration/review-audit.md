# 독립 검수 보고 (Review Audit) — impl 산출물

> Directive #014 First work item / review 노드(fork-sync~review) 독립 검수.
> 검수 대상: `notes/integration/patches/` (번들 20패치 + README), `notes/integration-workflow.md`.
> Ground truth: `notes/fork-changes-inventory.md` + `git diff 71ca7a1 HEAD -- src/` (20 src 파일, +1037/-26).
> [갱신 2026-06-07 #016] 이 보고 작성 당시 델타는 +1022/-26였으나, #015(`594a6e8`, gil_scaffold codegen fix)로 `gil_scaffold.ts` 누적 델타가 변해 현 권위 델타는 **+1037/-26**. C2 패치는 #016 baseline-refresh로 재생성됨(`notes/integration/baseline-refresh-audit.md` 참조). #014 verdict(20패치 바이트 일치·clean apply) 자체는 refresh 후에도 재검증으로 유효.
> 관점: 비판적·독립적. impl 자가평가를 신뢰하지 않고 소스에서 직접 재검증.
> 검수일: 2026-06-06. **워킹트리 변형 없음** (임시 worktree로만 검사, 검사 후 제거·prune 확인).

---

## 종합 판정: **통과 (PASS)**

근거 한 줄: 20개 패치 전부가 권위 델타(`git diff 71ca7a1 HEAD -- <file>`)와 **바이트 단위 동일**, 치명 버그수정 B1/B2/B5 hunk 직접 확인됨, 분기점 clean apply(개별+일괄) 검증됨. 발견 이슈는 전부 경미(presentation 수준)로 통합 진행을 막지 않음.

---

## 항목별 verdict (C1~C5)

### C1 완전성 — **PASS**
- 권위 stat의 20 src 파일이 번들 20 패치에 1:1 매핑됨 (검증: 파일별 diff 자동 대조, 아래 C2 표).
- 인벤토리 코드영향 항목 매핑:
  - A1~A13: A01~A07/A09~A12/A13B1/A8B5 패치에 전부 캡처. A8은 A8B5에 포함(nodes.ts 공유) — 정확.
  - B1→A13B1, B2→B2, B3→A02(core.ts 동일 hunk), B4→A11(index.ts 동일 hunk), B5→A8B5(hunk3). 전부 캡처.
  - C1~C6: C1~C6 패치 전부 캡처.
  - D1(.gitignore): **의도적 비포함**. 코드영향 0 + src 미해당 + `.claude/` 관리외. README/워크플로 추적표에 명시 기록 → 누락 아님, 합당.
- **누락 0건. 중복 0건.** (impl이 A8/A13의 파일공유를 단일 패치로 묶고 양쪽 인벤토리 항목에 동시 매핑한 처리가 정확.)
- 산술 주: 스펙의 "22항목" 표현은 인벤토리 item-label 기준(B1/B5를 별도 항목으로 셈)이고, 실제 src 파일은 20개(B1↔A13, B5↔A8 파일공유, D1=비-src). 번들 20 파일패치가 25 item-label(A1~13,B1~5,C1~6,D1 중 D1제외 24 코드 + D1 추적)을 정확히 커버. 불일치 아님 — 셈 기준 차이를 README가 명시함.

### C2 버그수정 무결성 (치명) — **PASS**
독립 재검증 방법: 각 패치를 `git diff 71ca7a1 HEAD -- <대상파일>`과 자동 대조 → **20/20 IDENTICAL (바이트 단위)**. 즉 패치 내용이 곧 커밋된 코드 델타 그 자체이므로, 약화/롤백/조건누락이 **구조적으로 불가능**. 추가로 치명 3건 hunk를 직접 grep 확인:
- **B1 varint overflow guard** (A13B1): 3함수 모두 보존 확인.
  - `while (offset >= 0 && offset < buf.length)` ×3 (readFieldMessages/readFieldBytes/parseNodeGraphId).
  - `if (len < 0) break` ×2, `dataEnd > buf.length || dataEnd < 0` ×2, `if (newOffset < 0) break` ×1.
  - 인벤토리 B1 설명과 의미 정확 일치. 가드 약화 흔적 없음.
- **B2 any 타입가드** (B2 패치): `+ if (type.flags & ts.TypeFlags.Any) return false` — isEntityLikeType 시작부, 1건 정확.
- **B5 parseValue entity fallback** (A8B5 hunk3): `entityLiteral` import + `z.union([z.int(), z.bigint()]).safeParse` → `return new entityLiteral(result.data)` 전부 보존.
- **A8B5 공존 처리**: nodes.ts 단일 파일에 A8(sendSignal)+B5(parseValue) 4 hunk 공존. impl이 물리 분리 불가(import 인접)를 정확히 진단하고 단일 패치 유지 + hunk 명세표 제공 + "hunk3 절대 드롭금지" 경고를 README/워크플로 양쪽에 기재. 무결성 보호 장치 적절.

### C3 충돌위험도 정확성 — **PASS (경미 이슈 1건)**
- README 패치표의 HIGH 라벨: A02(core), A8B5(nodes), A10(mappings), A11(index), A12(pins), A13B1(signal_nodes) = 인벤토리 HIGH-7과 일치.
- B5는 README에서 "MED(파일은 HIGH)"로 표기 — 인벤토리(B5=HIGH, parseValue 핫스팟)와 정합(파일 HIGH 명시).
- MED/LOW 라벨(A1,A3,A6,B1,B2,C4 / A4,A5,A7,A9,C1~C3,C5,C6) 인벤토리와 일치.
- **[경미 이슈 R1]** 워크플로 §2 표 제목이 "HIGH 7지점"인데 1행에 `runtime/value.ts`(A1)를 포함. A1은 인벤토리상 **MED**. 다만 해당 행이 인라인으로 "A1(MED, 선행 의존)"이라 명시 표기하고, 의존 순서(value.ts가 SignalArgDef export) 때문에 재적용 1순위로 넣은 것 — 오분류가 아니라 "HIGH 충돌파일 6 + 선행의존 1"을 7행으로 묶은 presentation. 혼동 여지만 있음. **권고**: 표 제목을 "재적용 순서(HIGH 6 + 선행의존 value.ts)"로 미세 수정하면 명확. 통합 진행 차단 아님.

### C4 재적용 가능성 — **PASS**
- 임시 worktree(`git worktree add --detach <tmp> 71ca7a1`)에서 검증:
  - 개별 `git apply --check`: **20/20 OK** (bad=0).
  - 일괄 `git apply --check *.patch`: **BATCH OK**.
- 검증 후 worktree remove --force + prune 완료. 실제 repo 트리 무변형 확인(tracked 수정 0건, worktree list 정상).
- 형식: 분기점 기준 파일별 unified diff. 계약이 허용한 "explicit hunk 문서/파일별 diff" 방식 — `git apply` 직접 소비 가능, format-patch보다 그룹 분할이 명확. 종속성 순서(value→core/compiler→injector) README/워크플로에 기재. 적합.

### C5 워크플로 실행가능성 — **PASS**
- 계약 6항목 전부 충족: ①머지전략(rebase 권장+근거표)+분기점 갱신절차(remote add~rebase --onto~번들재생성), ②HIGH 재적용 순서표+충돌 일반절차, ③proto upstream 채택 명시(+역분석 문서 혼동 경고), ④B1/B2/B5 롤백금지 체크리스트(grep 검증커맨드 포함), ⑤재적용 후 검증(npm install/tsc/build/grep가드/인젝션 스모크), ⑥상태 추적표(미적용/적용/검증완료).
- 절차가 구체적(실 명령 포함)이고 모호함 낮음. 버그체크리스트가 grep 기댓값(B1 "8건 이상" 등)까지 제시 — 실행가능성 높음.
- **[경미 이슈 R2]** §5 검증 명령(`npx tsc --noEmit`, `npm run build`)이 현재 package.json 실제 스크립트와 일치하는지 미확인 표기는 있으나("스크립트명 달라질 수 있음"), 현 시점 package.json 기준 정확 스크립트명을 부기하면 더 친절. 차단 아님.

---

## 발견 이슈 목록 (심각도)

| ID | 심각도 | 항목 | 내용 | 권고 |
|----|--------|------|------|------|
| R1 | LOW (presentation) | C3 | 워크플로 §2 "HIGH 7지점" 표에 MED인 value.ts(A1) 포함 — 인라인 MED 표기로 오분류는 아니나 제목이 오해 소지 | 표 제목 "재적용 순서(HIGH 6 + 선행의존)"로 미세 수정 |
| R2 | LOW (편의) | C5 | §5 검증 스크립트명이 현 package.json과 일치하는지 미확정(주석으로 면책됨) | 현 package.json scripts 기준 정확명 부기(선택) |

치명/중대(HIGH/MED severity) 이슈: **0건.** 누락/롤백/약화/오분류: **0건.**

---

## 독립 검증 로그 (재현 가능)
1. 파일별 권위 diff vs 패치 바이트 대조 (20/20 IDENTICAL).
2. 치명 hunk grep: B1(8+ 가드), B2(1), B5(import+fallback), A13(kind=5) 전부 확인.
3. 임시 worktree(71ca7a1) `git apply --check` 개별 20/20 + 일괄 OK.
4. worktree 제거·prune, repo tracked 수정 0건 확인.

---

## 자기평가 (directive 기준)
- C1 완전성: **PASS** (누락/중복 0, D1 의도적 비포함 합당).
- C2 버그수정 무결성: **PASS** (20/20 바이트 동일 → 약화 구조적 불가, B1/B2/B5 hunk 직접 확인).
- C3 충돌위험도: **PASS** (경미 R1 — 오분류 아닌 presentation).
- C4 재적용: **PASS** (worktree clean apply 개별+일괄, 트리 무변형).
- C5 워크플로: **PASS** (계약 6항목 충족, 경미 R2).
- 제약 준수: src/번들 미수정, 워킹트리 무변형(검사전용 worktree만 사용), `.claude/` 무시. **준수.**
- 한계: 정적 검증만 수행(빌드/타입체크/런타임 스모크는 upstream URL 미확보로 미실행 — 워크플로 §5에 절차로 위임됨, 현 단계 정상).

**종합: 통과 (PASS) — 조건 없음. R1/R2는 선택적 미세 개선 권고(차단 아님).**
