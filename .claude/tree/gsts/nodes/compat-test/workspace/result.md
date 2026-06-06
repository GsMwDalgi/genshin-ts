# GIL Compatibility Test Result (Corrected)

## Summary
**Status: INJECTION SUCCESS** — all 3 graphs compiled and injected into `1073741966.gil` without errors.

## Test Details

Replicated elementalist source files into `gsts-sandbox/src/elementalist-compat/`, targeting mapId `1073741966`:

| File | Target Graph ID | Target Name | Result |
|------|----------------|-------------|--------|
| character_inventory.ts | 1073741905 | _GSTS_character_inventory | OK (96ms) |
| compat_test.ts | 1073741906 | _GSTS_character_empty_01 | OK (28ms) |
| stage_droptable.ts | 1073741915 | _GSTS_stage_droptable | OK (48ms) |

## Pipeline Results

1. **TS -> GS IR compilation**: OK (4 files including db.ts)
2. **IR -> JSON**: OK (3 graphs)
3. **JSON -> GIA**: OK (3 .gia files generated)
4. **GIA -> GIL injection**: OK (3/3 injected, 0 failures)

## Observations

- No GIL format changes detected after game version update
- The protobuf schema, header format, and encoding are unchanged
- Signal node resolution works (request_droptable, request_internal_droptable, response_droptable, response_droptable_index all resolved)
- The `1073741966.gil` map has 175 node graphs including 10 `_GSTS_*_empty_*` placeholder slots
- `1073741967.gil` is a separate empty map (79 bytes, 0 node graphs) — not the target

## Note on Initial Confusion
First attempt targeted `1073741967.gil` (wrong file — empty map). After correction, `1073741966.gil` (the actual elementalist map) was used and injection succeeded.

## Files Created in gsts-sandbox
- `src/elementalist-compat/db.ts` — item database (exact copy from elementalist)
- `src/elementalist-compat/character_inventory.ts` — droppable management (id: 1073741905)
- `src/elementalist-compat/stage_droptable.ts` — 3-filter droptable (id: 1073741915)
- `src/elementalist-compat/compat_test.ts` — minimal test graph (id: 1073741906)
- Config restored to original (SampleForEffect, mapId 1073741960)
