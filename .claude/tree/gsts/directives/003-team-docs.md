# Directive #003
- Domain: documentation
- Summary: Write teammate installation and setup guide for gsts-sandbox and elementalist
- Created: 2026-03-31
---

## Goal

Create a document that a teammate can follow to set up their development environment for gsts-sandbox and/or elementalist projects.

## Requirements

1. **Location**: `notes/team-setup-guide.md` in the genshin-ts project

2. **Contents must cover**:
   - Prerequisites (Node.js, npm, git)
   - How to clone the forked genshin-ts repo
   - How to run `npm install` and `npm link` in genshin-ts
   - How to set up gsts-sandbox or elementalist (clone, npm install, npm link genshin-ts)
   - How to configure their own UID and map file name in `gsts.config.ts`
   - How to run build/dev commands
   - Warning about not mixing up UID/map files between users

3. **UID/map confusion prevention**:
   - Document that each user MUST set their own UID and map file path in `gsts.config.ts`
   - Suggest using environment variables or `.env` files for per-user settings (investigate what gsts.config.ts supports)
   - If `.env` is feasible, add `.env` to `.gitignore` in each project template

4. **Language**: Korean, technical terms in English

## Dependencies

- Directive #002 (project setup) should be completed first or in parallel, so the guide reflects actual setup steps.
- Read `create-genshin-ts/templates/start/README.md` and `gsts.config.ts` to understand the actual config structure.
