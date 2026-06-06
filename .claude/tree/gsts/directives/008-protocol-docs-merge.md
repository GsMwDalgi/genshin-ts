# Directive #008
- Domain: documentation/protocol
- Summary: Merge two protocol doc sets into unified notes/protocol/
- Created: 2026-03-31
---

## Goal

Merge two sources of GIA/GIL protocol documentation into a single unified set under `notes/protocol/`.

## Sources

1. **Primary (authoritative)**: `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\gia-protocol/` — 14 files, Korean, detailed analysis with confidence levels (CONFIRMED/INFERRED/SPECULATED), based on binary comparison
2. **Secondary**: `notes/protocol/` — 5 files, English summary, independently written

## Conflict Resolution

When information conflicts between the two sources, **gia-protocol takes priority** — it is the original analysis based on actual binary verification.

## Node Structure (user-specified)

- **root~doc-struct** (opus): Analyze both doc sets, plan unified structure, determine file layout and merge strategy
- **root~doc-writer** (opus): Write the unified documents based on the structure plan
- **root~doc-review** (opus): Review merged output for:
  1. **Accuracy** — no contradictions, confidence levels correct
  2. **Completeness** — no information lost from either source
  3. **Readability** — someone new to the protocol can follow the docs sequentially

## Output

Unified docs in `notes/protocol/` replacing the current 5 files. Preserve the confidence level system from gia-protocol.

## References

- Source 1: `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\gia-protocol/` (14 files)
- Source 2: `notes/protocol/` (5 files, current)
