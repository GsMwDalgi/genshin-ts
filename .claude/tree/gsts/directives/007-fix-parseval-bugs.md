# Directive #007
- Domain: toolchain/bugfix
- Summary: Fix parseValue and str() type conversion bugs blocking compilation
- Created: 2026-03-31
---

## Context

Two bugs prevent certain test files from compiling to GIA. These bugs exist in upstream genshin-ts (confirmed identical behavior in legacy). Fix them in the fork.

## Bug 1: `str()` unsupported input type

- **Error**: `[error] str(): unsupported input type` at `src/runtime/server_globals.ts:229`
- **Trigger**: Signal handler receives a signal argument and tries to convert it via `str()`. The value passed is not a raw string or `str` instance — it's likely a signal argument wrapper that `parseValue` and `convertIfNeeded` don't handle.
- **Failing test**: `test-compile-func/04-timers-signals-utils.ts` (line 56 in compiled .gs.ts), also `test-signal-args/signal_args_test.ts`
- **Flow**: `server_globals.ts` line 221-229 — `parseValue` throws, `convertIfNeeded` returns null, then throws "unsupported input type"

## Bug 2: `entity` type in parseValue

- **Error**: `Invalid value type: entity` at `src/definitions/nodes.ts:401`
- **Trigger**: `setCustomVariable` is called with an entity value. `parseValue(v, 'entity')` only checks `instanceof entity` — unlike `guid`, `prefab_id`, `config_id` which have bigint fallback, `entity` has no fallback path and falls through to throw.
- **Failing test**: `SampleForEffect/EffectViewer.ts` — `setCustomVariable(evt.eventSourceEntity, 'myVar', 42n)` where the first arg triggers entity type parsing.

## Acceptance Criteria

1. `npm run build` with entries `['./src/test-compile-func/']` completes with 0 errors (all 18 files including 04)
2. `npm run build` with entries `['./src/SampleForEffect/']` completes with 0 errors
3. No regressions — existing passing tests must still pass

## Test Projects

- `D:\MyDrive\Repos\MiliastraWonderland\gsts-sandbox` (npm link to this genshin-ts)
- Config: change `entries` to target the relevant folder, then `npm run build`

## References

- `src/runtime/server_globals.ts` lines 220-232 (str conversion)
- `src/definitions/nodes.ts` lines 237-242 (entity case in parseValue)
- `src/definitions/nodes.ts` line 401 (throw point)
- Legacy (identical code): `D:\MyDrive\Repos\MiliastraWonderland\legacy\genshin-ts`
