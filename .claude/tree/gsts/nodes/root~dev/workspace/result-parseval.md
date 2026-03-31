# Result: Fix parseValue and str() type conversion bugs (Directive #007)

## Status: Complete

## Changes Made

### Bug 1: `str()` unsupported input type — fixed by adding signal args support to `onSignal`

The root cause was that `onSignal` didn't accept signal arg definitions. When a signal handler declared custom args (e.g., `{ name: 'count', type: 'int' }`), they were ignored, so `evt.count` was `undefined`, and `str(undefined)` threw "unsupported input type".

**Files changed:**
- `src/runtime/core.ts` — Updated `ServerGraphApi.onSignal()` interface and implementation to accept optional `SignalArgDef[]` third parameter. Updated `runHandler` and `runServerHandler` to propagate signal args. Updated `registerEvent` to create additional output pins for signal args on the event node, with proper `signalParams` metadata for IR building.

### Bug 2: `entity` type in parseValue — fixed by adding bigint fallback

The `entity` case in `parseValue` only checked `instanceof entity` with no fallback for bigint/number values, unlike `guid`, `prefab_id`, `config_id` which all had `z.union([z.int(), z.bigint()])` fallback paths.

**Files changed:**
- `src/definitions/nodes.ts` — Added `entityLiteral` to imports. Added bigint/int fallback to the `entity` case in `parseValue`, creating `new entityLiteral(result.data)` (matches the pattern of other types).

### GIA transform: monitor_signal output pins for signal args

- `src/compiler/ir_to_gia_transform/index.ts` — Added import for `SIGNAL_ARG_TYPE_MAP`. Added output pin generation (kind=4) for `monitor_signal` event nodes with `signalParams`, using correct pin indices offset by the 3 standard monitorSignal outputs.

## Test Results

1. `npm run build` with entries `['./src/test-compile-func/']` — **18/18 files compiled, 18/18 GIA generated, 0 errors**
2. `npm run build` with entries `['./src/SampleForEffect/']` — **2/2 files compiled, 2/2 GIA generated, 0 errors**
3. genshin-ts itself builds with `tsc` — **0 errors**
