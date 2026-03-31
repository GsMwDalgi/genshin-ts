# Result: Team Setup Guide (Directive #003)

## What Was Written

- **File**: `notes/team-setup-guide.md` in the genshin-ts project root
- **Language**: Korean with technical terms in English
- **Sections**:
  1. Prerequisites (Node.js v20+, npm, git)
  2. Step 1 — Clone and npm link genshin-ts fork
  3. Step 2 — Set up user project (gsts-sandbox or elementalist)
  4. Step 3 — gsts.config.ts inject configuration (playerId, mapId, gameRegion)
  5. Step 4 — Build and dev commands
  6. UID/MapId confusion prevention section with checklist and two approaches

## Config Structure Findings

`GstsConfig` (from `src/compiler/gsts_config.ts`):
- `inject.playerId` — player UID (numeric folder name under BeyondLocal)
- `inject.mapId` — target map file ID
- `inject.gameRegion` — 'China' or 'Global'

These are the per-user fields that must differ between teammates.

## .env / process.env Feasibility

**Verdict: feasible but not automatic.**

- `gsts.config.ts` is executed via tsx subprocess. It inherits the parent process's environment, so `process.env` variables ARE available.
- tsx does NOT auto-load `.env` files.
- Node 20+ `--env-file` flag is not passed through the gsts CLI's tsx invocation.
- `.env` is already in the template's `.gitignore`, so the `.env` approach is anticipated.

**Recommended approach**: Use `process.env` in `gsts.config.ts`, set variables in shell profile or system env. Documented as "Method B" in the guide.

**Simplest approach**: Each user edits `gsts.config.ts` directly on their own clone and keeps it out of git via `.gitignore`. Documented as "Method A" with `gsts.config.example.ts` pattern.

## No Changes Made to Existing Files

The `.gitignore` in both gsts-sandbox and elementalist already contains `.env` (from the template). No modifications needed.

## Status

Complete.
