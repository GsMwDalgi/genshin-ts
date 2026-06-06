---
domain: integration/upstream-merge
---
# Directive #018 — Pre-promotion Bug-possibility Audit

- Summary: Adversarial, independent bug hunt on the #017 merge result BEFORE promoting to master.
- Created: 2026-06-07

## Context

The #017 upstream merge (direction B) is committed on branch **`integ-upstream` @ `d2864d9`** (master still `8629dc9`, backup tag `fork-backup-20260607`). Prior review PASSed (§4 bugfix grep, §5 tsc/build/smoke). Verdict: `.claude/tree/gsts/nodes/fork-sync/workspace/verdict-017.md`. Full audit: `notes/integration/upstream-merge-audit.md`. The user wants a fresh independent bug check before the (force, non-ff) master promotion.

## Goal

Find ANY correctness bug introduced by the merge. Default to skepticism — assume something was dropped or mis-ported until proven otherwise. Produce a clear go/no-go for promotion.

## Focus areas (verify on integ-upstream @ d2864d9)

1. **B1/B2/B5 semantic correctness (not just presence)** — the guards were re-ported onto upstream's restructured code (B1 now spans `injector/binary.ts` + `injector/signal_nodes.ts` because upstream moved `readFieldBytes`/`readFieldMessages` to binary.ts). Confirm each guard is in the ACTUAL hot path and actually prevents its failure (B1 varint overflow/infinite-loop; B2 `any`→entity misinference; B5 parseValue entity fallback). Diff against upstream original to confirm they are our additions, not upstream's.
2. **Group-A removal completeness** — `git grep` for residue (signalArgs/SignalArgDef/SignalArgsToPayload/SIGNAL_ARG_TYPE_MAP) = 0, AND no dangling references, broken imports, dead code, or broken call sites from discarding signal-args. Nothing should reference the removed API.
3. **Signal end-to-end (close verdict §9-1 gap)** — attempt to complete the pin-depth check that was skipped (fixture missing): compile a `defineSignal` sample → inject/transform → verify GIA placeholder pins (`send_signal:300000` / `monitor_signal:300001`) and typed `evt.params` resolve correctly end-to-end. If the fixture genuinely cannot be created, state exactly why and what residual risk remains.
4. **C CLI regression** — `gsts inspect` / `gsts scaffold` on the sample `.gil` (`1073741976.gil`, id `1073741920`) against the new upstream base; confirm no behavioral break and #015 scaffold fix preserved.
5. **Full `npm test` triage** — run it; for EVERY failure, determine whether it reproduces on a pure upstream worktree (= pre-existing upstream bug, acceptable) or is introduced by our merge (= blocker). List each failure with its classification.
6. **Build sanity** — `npm run build` produces dist + proto/d.ts copy; tsc --noEmit clean.

## Output

Inline summary to leader: findings list each tagged **[blocker] / [major] / [minor] / [info]**, then an explicit verdict line: **"SAFE TO PROMOTE"** or **"NOT SAFE — <reasons>"**. Reference evidence (commands + EXIT codes). If a deeper artifact is written, put it under `notes/integration/`.

## Constraints

- Read-only on source (no fixes — report only). Do not touch `master`. Do not pollute repo root (use ramdisk R: / temp worktrees, clean up after).
- Independent verification — re-run checks yourself; do not trust prior impl/review logs at face value.
