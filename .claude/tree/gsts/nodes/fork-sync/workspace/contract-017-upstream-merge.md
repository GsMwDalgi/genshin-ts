# Contract #017 — Upstream Merge Execution

> 공유 계약. impl(fork-sync~impl)과 review(fork-sync~review)가 모두 준수.
> Directive: `.claude/tree/gsts/directives/017-upstream-merge.md`
> 권위 절차: `notes/integration-workflow.md` §1~§5 + 부록.
> 분류 참조: `notes/fork-changes-inventory.md`. 번들: `notes/integration/patches/` (20패치).
> Mode: autonomous. Model: opus.

## upstream
- URL: `https://github.com/josStorer/genshin-ts`
- 우리 분기점: `71ca7a1` (선형). 현 HEAD: `8629dc9` (leader가 #014~#016 베이스라인 커밋).
- 권위 델타: `git diff 71ca7a1 HEAD -- src/` (20 src 파일 = 번들 20패치와 정합).

## 산출물 경로 (고정)
| 산출물 | 경로 | 소유 |
|--------|------|------|
| impl 결과 | `.claude/tree/gsts/nodes/fork-sync~impl/workspace/result-017.md` | impl |
| review audit | `notes/integration/upstream-merge-audit.md` | review |
| merge log (선택) | `.claude/tree/gsts/nodes/fork-sync~impl/workspace/merge-log-017.md` | impl |
| 통합 verdict | `.claude/tree/gsts/nodes/fork-sync/workspace/verdict-017.md` | fork-sync |

## 브랜치/태그 규약 (불변 — reversibility)
- 백업 태그: `fork-backup-YYYYMMDD` = 현 HEAD(`8629dc9`)에 **먼저** 생성.
- 작업 브랜치: `integ-upstream` (또는 명확한 전용 브랜치명). 머지는 여기서.
- **master를 force-update 하지 말 것.** 검증 통과 + leader 수용 전까지 master 불변. 작업은 전부 브랜치+태그로 가역.
- 루트 워킹트리 더럽히지 말 것. scratch는 temp worktree / R: 램디스크.

## 수용 기준 (impl/review 공통 판정축)

### G0 — NEW_BASE 판정 (게이트)
- `git fetch upstream` 후 NEW_BASE = upstream tip 해시 기록.
- **NEW_BASE == 71ca7a1 이면**: upstream 미전진 → 머지할 것 없음. 그 사실+근거(해시 동일)만 보고하고 중단. G1~G5 N/A.
- NEW_BASE != 71ca7a1 이면 머지 진행 → G1~G5 적용.

### G1 — 브랜치+태그 가역성
- 백업 태그가 옛 HEAD(8629dc9)에 존재. 작업 브랜치 분리됨. master 미변경(`git rev-parse master` == 8629dc9 유지).

### G2 — HIGH-7 충돌 해소 (순서: value.ts → core.ts → nodes.ts → mappings.ts → index.ts → pins.ts → signal_nodes.ts)
- 각 파일별: 충돌 여부 / 해소 방식(우리 기능 upstream 새 구조 위 재구현 vs 변형폐기) 기재.
- signal-args 기능 보존(삭제·우회 금지). upstream이 동등기능 자체도입 시 → 부록 기준 + review 독립확인 후만 폐기.

### G3 — B1/B2/B5 무결성 (§4, 절대 롤백금지)
grep 증거(impl 실행 + review 독립 재실행):
- B1: `grep -nE "offset >= 0|len < 0|dataEnd < 0|newOffset < 0" src/injector/signal_nodes.ts` → 8건 이상.
- B2: `grep -n "TypeFlags.Any) return false" src/shared/ts_type_utils.ts` → 1건.
- B5: `grep -n "new entityLiteral" src/definitions/nodes.ts` → 1건 이상.

### G4 — §5 검증 (구체적 증거 필수, EXIT 코드 포함)
- `npm install` ok.
- `npx tsc --noEmit` (또는 package.json typecheck 스크립트) EXIT=0.
- build ok (`npm run build` 또는 정의된 스크립트).
- §4 grep 3종 present (G3와 동일).
- injection smoke: signal-args end-to-end (sendSignal/onSignal+signalArgs 샘플 → GIA 핀 생성) + CLI 회귀 `gsts inspect`/`gsts scaffold` on 샘플 .gil `1073741976.gil`.
  - 샘플 .gil: `C:\Users\Rterg\AppData\LocalLow\miHoYo\Genshin Impact\BeyondLocal\804101570\Beyond_Local_Save_Level\1073741976.gil`.

### G5 — 변형폐기 결정 (해당 시)
- 폐기한 우리 변형이 있으면: 무엇을/왜(upstream 동등기능 도입) + 부록 동등성 기준 충족 + **review 독립확인** 명시. 폐기 없으면 "없음".

## 보고 형식 (verdict-017이 leader에 전달해야 할 항목 — directive §Deliver)
- 브랜치명 + 백업 태그명.
- NEW_BASE 해시.
- HIGH-7별 충돌해소 결과.
- §4 체크리스트 결과(grep 카운트).
- §5 검증 증거(EXIT 코드/로그).
- 변형폐기 결정(review 확인됨).

## 분할 (Org 제약 — impl≠review)
- impl: G0~G5 실행. result-017.md에 G0~G5 self-claim + 증거.
- review: G3/G4 grep·검증 **독립 재실행**(impl 로그 신뢰 금지), G2 충돌해소 정합성, G5 변형폐기 독립확인. upstream-merge-audit.md.
- fork-sync: review PASS 후 verdict-017 통합 → leader. master force는 leader 수용 후 별도.
