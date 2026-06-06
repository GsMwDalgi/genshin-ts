# Directive #002
- Domain: tooling/setup
- Summary: Set up gsts-sandbox and elementalist projects with npm link to local genshin-ts
- Created: 2026-03-31
---

## Goal

Create two user projects that use the locally forked genshin-ts:
1. **gsts-sandbox** — test/validation project at `D:\MyDrive\Repos\MiliastraWonderland\gsts-sandbox`
2. **elementalist** — production project at `D:\MyDrive\Repos\MiliastraWonderland\elementalist` (folder already exists, empty)

## Requirements

### For both projects:

1. Initialize using the genshin-ts template (`npm create genshin-ts@latest` or copy from `create-genshin-ts/templates/start/`)
2. Link to the local forked genshin-ts via `npm link`:
   - First, in `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts`, run `npm link` (if not already done)
   - Then in each project, run `npm link genshin-ts`
3. Verify `npm run build` works in each project

### gsts-sandbox specific:
- This is for testing genshin-ts features — keep the default template entry as-is for build verification

### elementalist specific:
- This is the production project — set up clean but do not add any game logic yet

## Notes

- The user and a teammate will both use these projects with different UIDs and map files. UID/map separation will be handled in a separate directive (docs).
- genshin-ts may already have `npm link` registered from previous setup with genshin-ts-run project. Verify and re-run if needed.
