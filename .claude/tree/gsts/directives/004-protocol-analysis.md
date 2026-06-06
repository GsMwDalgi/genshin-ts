# Directive #004
- Domain: research/protocol
- Summary: Analyze GIA/GIL file protocols and store raw findings in notes/protocol/
- Created: 2026-03-31
---

## Goal

Analyze GIA and GIL (map) file formats by comparing generated outputs with existing map files. Store raw analysis results for future use by the mw-editor project.

## Requirements

1. **Output location**: `notes/protocol/` in the genshin-ts project

2. **Analysis sources**:
   - Generated `.gia` files from `dist/` after a build
   - The thirdparty code at `src/thirdparty/Genshin-Impact-Miliastra-Wonderland-Code-Node-Editor-Pack/` — contains reverse-engineered format handling
   - Any existing map files accessible from the project or genshin-ts-run (`D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\`)
   - Source code in `src/` that handles file reading/writing (injector, compiler output)

3. **What to document**:
   - File structure (headers, sections, data layout)
   - Field types and meanings
   - Differences between GIA and GIL formats
   - How genshin-ts generates/injects into these files
   - Any magic numbers, version fields, checksums

4. **Quality level**: Raw data collection, not polished documentation. Organize files logically but don't spend time on formatting or prose. Code snippets, hex dumps, field tables are all fine.

5. **File naming**: Use descriptive names like `gia-structure.md`, `gil-structure.md`, `injection-flow.md`, etc.

## Notes

- This is pre-work for a future editor project (mw-editor). The refined documentation will be done there.
- Commit-ready state is fine — the user wants these tracked in git.
