# Task: Fix parseValue and str() type conversion bugs (Directive #007)

## Directive detail
See `.claude/tree/gsts/directives/007-fix-parseval-bugs.md`

## Bug 1: `str()` unsupported input type

- **Error**: `[error] str(): unsupported input type` at `src/runtime/server_globals.ts:229`
- **Trigger**: Signal handler receives a signal argument and tries to convert it via `str()`. The value passed is not a raw string or `str` instance — it's likely a signal argument wrapper that `parseValue` and `convertIfNeeded` don't handle.
- **Failing test**: `test-compile-func/04-timers-signals-utils.ts`, also `test-signal-args/signal_args_test.ts`
- **Flow**: `server_globals.ts` line 221-229 — `parseValue` throws, `convertIfNeeded` returns null, then throws "unsupported input type"

## Bug 2: `entity` type in parseValue

- **Error**: `Invalid value type: entity` at `src/definitions/nodes.ts:401`
- **Trigger**: `setCustomVariable` called with an entity value. `parseValue(v, 'entity')` only checks `instanceof entity` — unlike `guid`, `prefab_id`, `config_id` which have bigint fallback, `entity` has no fallback path and falls through to throw.
- **Failing test**: `SampleForEffect/EffectViewer.ts`

## Key source files
- `src/runtime/server_globals.ts` lines 220-232 (str conversion)
- `src/definitions/nodes.ts` lines 237-242 (entity case in parseValue)
- `src/definitions/nodes.ts` line 401 (throw point)

## Testing

Use gsts-sandbox (`D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox`) which is npm-linked to genshin-ts:
1. Change `entries` in gsts.config.ts to `['./src/test-compile-func/']`, run `npm run build` — expect 0 errors (all 18 files including 04)
2. Change `entries` to `['./src/SampleForEffect/']`, run `npm run build` — expect 0 errors
3. Verify no regressions with existing passing entries

## Acceptance Criteria
1. `npm run build` with entries `['./src/test-compile-func/']` completes with 0 errors
2. `npm run build` with entries `['./src/SampleForEffect/']` completes with 0 errors
3. No regressions

## Output
Write completion report to your `workspace/result-parseval.md` with what was changed and test results.
