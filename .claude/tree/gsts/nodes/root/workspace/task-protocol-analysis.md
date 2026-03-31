# Task: Analyze GIA/GIL file protocols

## Context
Analyze GIA and GIL (map) file formats by reading source code and comparing with generated outputs. This is pre-work for a future editor project (mw-editor).

## Output location
Write all analysis files to `D:/MyDrive/Repos/MiliastraWonderland/genshin-ts/notes/protocol/` (create directory if needed).

## Analysis sources (read these)
1. Thirdparty code: `src/thirdparty/Genshin-Impact-Miliastra-Wonderland-Code-Node-Editor-Pack/` — contains reverse-engineered format handling
2. Source code in `src/` that handles file reading/writing (injector, compiler output)
3. Generated `.gia` files from `dist/` (if any exist after build)
4. Any existing map files in the project or in `D:/MyDrive/Repos/MiliastraWonderland/genshin-ts-run/`

## What to document
- File structure (headers, sections, data layout)
- Field types and meanings
- Differences between GIA and GIL formats
- How genshin-ts generates/injects into these files
- Any magic numbers, version fields, checksums
- Injection flow (how code gets injected into map files)

## Quality level
Raw data collection, not polished documentation. Code snippets, hex dumps, field tables are all fine. Organize files logically.

## File naming
Use descriptive names: `gia-structure.md`, `gil-structure.md`, `injection-flow.md`, etc.

## Output
Write a brief completion report to your `workspace/result.md` listing what files were created and key findings.
