# Result: Team Setup Guide Revision (Directive #005)

## Changes Made

Rewrote `notes/team-setup-guide.md` from scratch targeting a gsts end-user.

**Removed:**
- Entire "genshin-ts fork 클론 및 설정" step (npm link, npm ls -g --link, toolchain internals)
- "새로 만드는 경우 (template에서 복사)" path — end users only clone an existing project
- `npm link genshin-ts` step
- `npm run typecheck` / `npm run lint` commands
- process.env / Method B approach (too technical for end users)
- Internal references (`node_modules/genshin-ts/src/`, definitions folder)

**Kept and improved:**
- Prerequisites (simplified)
- Clone → npm install → configure gsts.config.ts → build/dev flow
- UID/map confusion prevention — made more prominent with ⚠️ heading
- .gitignore recommendation for gsts.config.ts
- mapId lookup via `npm run maps`

**Structure of revised guide:**
1. Prerequisites
2. Clone project
3. Personal settings (UID · mapId) — inject block, how to find values
4. UID/map confusion prevention (prominent, with checklist)
5. Build and dev commands

## Acceptance Criteria Check
- Clear enough for non-developer: yes — no toolchain concepts exposed
- No toolchain internals: yes — all genshin-ts/npm link content removed
- UID/map confusion prevention prominent: yes — dedicated section with warning and checklist

## Status
Complete. Ready for readability review.
