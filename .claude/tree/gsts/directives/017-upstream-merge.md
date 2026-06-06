---
role: fork-maintenance
domain: integration/upstream-merge
---
# Directive #017 — Upstream Merge Execution

- Summary: Execute the upstream merge now that the original repo URL is available.
- Created: 2026-06-07
- upstream URL: https://github.com/josStorer/genshin-ts

## Goal

Integrate the latest upstream `josStorer/genshin-ts` into our fork while honoring the 3-axis mission:
1. **우리 변형 유지** — signal-args (group A, HIGH 7 hotspots), CLI/Reader (group C).
2. **upstream 변형 최소화** — keep our code as close to upstream form as possible; linear history.
3. **통합 상태 지속 유지** — fork remains mergeable, baseline bundle stays the authority.

## Authoritative procedure

`notes/integration-workflow.md` is the procedure of record. Follow it:
- §1 strategy + branch-point update (backup tag first)
- §2 HIGH-7 conflict resolution order (value.ts → core.ts → nodes.ts → mappings.ts → index.ts → pins.ts → signal_nodes.ts)
- §3 proto adoption (upstream as-is)
- §4 **B1/B2/B5 integrity checklist — 절대 롤백 금지**
- §5 verification (install / tsc / build / bugfix-guard greps / injection smoke)

Delta classification reference: `notes/fork-changes-inventory.md`. Patch bundle: `notes/integration/patches/` (20 patches, branch-point clean-apply verified, == current HEAD authority delta).

## Constraints

- **Reversibility (robustness)**: do the merge on a dedicated branch (e.g. `integ-upstream`) with a backup tag (`fork-backup-YYYYMMDD HEAD`) created first. **Do NOT force-update `master` until verified AND accepted.** The branch + tag keep the operation fully reversible.
- **Branch-point check first**: fetch upstream, find NEW_BASE (upstream tip). If `NEW_BASE == 71ca7a1` (upstream has not advanced past our fork point), there is nothing to merge — report and stop.
- proto / gia_gen / thirdparty → adopt upstream as-is (we have 0 changes there).
- **B1/B2/B5 bugfix integrity** (varint guard / any type-guard / parseValue entity fallback) must survive — review verifies via §4 grep evidence independently. Never silently dropped during conflict resolution.
- **Variation-discard rule (mission axis-1 exception)**: if upstream has independently introduced equivalent signal-args / bugfixes, apply `notes/integration-workflow.md` 부록 criteria; review must independently confirm equivalence BEFORE any of our variations are discarded.
- **impl ≠ review split** — implementation and independent audit are separate nodes (no self-review).
- Keep the repo root clean; use temp worktrees / ramdisk (R:) for scratch as established.

## Verification gates (§5)

Report concrete evidence for each: `npm install` ok, `tsc --noEmit` EXIT=0, build ok, the 3 bugfix-guard greps present, injection smoke (signal-args end-to-end GIA pin generation + `gsts inspect`/`scaffold` regression on the sample .gil `1073741976.gil`).

## user constraint

- Mode: autonomous. Model: opus (org standard).
- Deliver a fork-sync verdict to leader with: branch name + backup tag, NEW_BASE hash, conflict-resolution outcome per HIGH-7, §4 checklist results (grep counts), §5 verification evidence, and any variation-discard decisions (review-confirmed).

## Leader Decision — signal-args direction (2026-06-07, autonomous + user-confirmed)

upstream `v0.1.10` (NEW_BASE `9cb31c8`) independently implemented an equivalent signal-args system (`defineSignal()` + typed `evt.params`) — parallel evolution, not adoption of ours. Mission axis-1 exception fired; user confirmed direction **B**.

- **Direction = B (discard + adopt upstream)**: discard our group A (A1~A13), adopt upstream `defineSignal()`/typed `evt.params`. Aligns with the documented mission strategy ("upstream 동일기능 도입 시 우리 변형 버리고 채택, 변형 최소화").
- **Constant across the merge** (re-confirmed): preserve **B1/B2/B5** — re-port onto upstream's new `signal_nodes` structure, §4 절대 롤백 금지; keep **C CLI** (resolve the gsts.ts registration conflict; upstream `gil_signals.ts` is a different file, no code conflict); adopt upstream **proto/gia_gen/thirdparty** as-is.
- **Equivalence gate (APPROVED → delegate to review)**: before any discard is finalized, the **review node independently verifies** upstream `defineSignal` covers (1) the 18-type coverage and (2) `_list` auto-wrap (workflow 부록 criteria). **Any gap → patch only the missing capability via C-hybrid** (no loss of our function). Discard is finalized only on review-confirmed equivalence (Org constraint: 폐기 결정은 review 독립검증 후).
- **Migration impact (ACCEPTED)**: downstream signal API consumers (gsts-sandbox / team git installs / mwe) migrate to the upstream API. The verdict MUST document the migration delta (our `sendSignal(...signalArgs)` / `onSignal<Args>` / `SIGNAL_ARG_TYPE_MAP` → upstream equivalent) so downstream can follow.
- **Reversibility checkpoint**: continue on `integ-upstream`; do **NOT** force-update `master` until review PASS + §5 verification + leader acceptance. Leader surfaces the final master-update with migration scope at acceptance.

fork-sync next: update `spec-impl-017` STEP2 to "discard A + adopt upstream defineSignal (+ C-patch any equivalence gap)", redispatch impl; then dispatch review to independently confirm equivalence + B1/B2/B5 integrity (§4) + §5 verification.

## RESUME — migration + verdict (leader, 2026-06-07, gen 3)

**Skill node-name migration applied.** The runtime now rejects `~` in teammate names; the separator is `_` and the node name doubles as the native handle. Your children were renamed on disk:
- `fork-sync~impl` → **`fork-sync_impl`**
- `fork-sync~review` → **`fork-sync_review`**

Use the `_` forms in ALL tree-cli.py calls and routing from now on (your prior memo/specs say `~` — treat those as the old names). Your own name `fork-sync` is unchanged.

**Both impl and review for #017 are DONE; their TASK_RESULTs are in your inbox** (impl `result-017.md`; review `result-review-017.md`, PASS / no blockers, full audit `notes/integration/upstream-merge-audit.md`). The earlier review→fork-sync routing failed (the `~` bug) and has been re-delivered.

**Your task now**: fold review's audit, do your independent re-confirm (per org), write `workspace/verdict-017.md`, and report TASK_RESULT to leader with the items listed under "user constraint" above (branch+tag, NEW_BASE, HIGH-7 outcome, §4 grep counts, §5 evidence, equivalence-gate result, migration delta). If review flagged any equivalence gap needing a C-patch, request `fork-sync_impl` activation via SPAWN_REQUEST; otherwise proceed straight to verdict. Update your memo to the `_` names.
