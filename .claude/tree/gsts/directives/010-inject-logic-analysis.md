# Directive #010
- Domain: injector/analysis
- Summary: Analyze current injection logic by cross-referencing docs and source code
- Created: 2026-04-09
---

## Context

Game version updated and GIL format may have changed. Need a thorough understanding of the current injection logic before any modifications.

## Task

1. Read the existing documentation:
   - `notes/protocol/injection-flow.md`
   - `notes/architecture/05-injector.md`
2. Read the actual injector source code in `src/injector/` (all files)
3. Cross-reference docs vs source — identify any gaps, outdated info, or undocumented behavior
4. Produce a comprehensive analysis document covering:
   - GIL file read/parse flow (binary format handling)
   - How GIA data is injected into GIL
   - GIL file write/output flow
   - Key data structures and their relationships
   - Any hardcoded assumptions about format (offsets, magic bytes, version fields, etc.)
   - Points that would break if the binary format changes
5. Save the analysis to your workspace as `inject-analysis.md`

## References

- `notes/protocol/injection-flow.md` — injection flow protocol doc
- `notes/architecture/05-injector.md` — injector architecture doc
- `notes/protocol/gil-binary-format.md` — GIL binary format doc
- `notes/protocol/gia-binary-format.md` — GIA binary format doc
- `src/injector/` — actual injector source code
