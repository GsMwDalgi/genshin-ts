# Memo

## Task
- TASK_ASSIGN — Directive #010: Analyze injection logic, cross-ref docs vs source (received 2026-04-09, complete)
- TASK_ASSIGN — Directive #011: Update inject docs with analysis findings + create mw-editor reference (received 2026-04-09, complete)

## Refs
- ../../directives/011-inject-docs-update.md | task directive (complete)
- workspace/inject-analysis.md | analysis output from #010
- notes/protocol/injection-flow.md | updated with gaps
- notes/architecture/05-injector.md | updated with gaps
- notes/protocol/injection-reference-for-mw-editor.md | new mw-editor reference doc

## Notes
- Part 1: Updated injection-flow.md (validation behavior, depth limit, dual-path resolution, fallback chain, proto caching, reader.ts section, sendServer kind, sources list). Updated 05-injector.md (parseMessage depth, dual-path resolution, type resolution fallback, sendServer kind, proto caching section, header validation section, depth limit note).
- Part 2: Created 13-section self-contained reference covering full pipeline, binary format, data structures, signal patching, graph type mapping, hardcoded values with file:line references.
