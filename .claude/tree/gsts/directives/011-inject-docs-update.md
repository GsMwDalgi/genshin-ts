# Directive #011
- Domain: injector/documentation
- Summary: Update existing inject docs with analysis findings and create mw-editor reference doc
- Created: 2026-04-09
---

## Context

Directive #010 analysis identified 8 documentation gaps and critical format-change risk areas. Additionally, mw-editor project's inject approach is broken and needs a reference document based on genshin-ts's working injection pipeline.

## Task

### Part 1: Update existing docs

Cross-reference your analysis (`nodes/inject-analyst/workspace/inject-analysis.md`) against:
- `notes/protocol/injection-flow.md`
- `notes/architecture/05-injector.md`

Add any missing information identified in your analysis, including but not limited to:
- file_type/schema_version validation behavior
- dual-path folder resolution
- sendServer kind documentation
- parseMessage depth=6 limit
- type resolution fallback chain
- reader.ts role in the flow
- proto caching behavior
- Any other gaps found

Preserve existing content — only add, don't restructure or rewrite.

### Part 2: Create mw-editor injection reference

Create a new document at `notes/protocol/injection-reference-for-mw-editor.md` that explains:
- The complete injection pipeline (read GIL → parse → inject GIA → write GIL)
- Binary format details (header structure, magic bytes, protobuf paths)
- Key data structures and their relationships
- Signal node patching mechanism
- Graph type mapping
- Hardcoded values and assumptions
- Step-by-step flow with file:line references to genshin-ts source

This document should be self-contained enough for another project (mw-editor) to understand and reimplement the injection logic. Write in English with Korean annotations where helpful.

## References

- `nodes/inject-analyst/workspace/inject-analysis.md` — your analysis output
- `notes/protocol/injection-flow.md`
- `notes/architecture/05-injector.md`
- `notes/protocol/gil-binary-format.md`
- `notes/protocol/gia-binary-format.md`
- `src/injector/` — source code
