# Task: Set up gsts-sandbox and elementalist projects

## Context
Create two user projects that use the locally forked genshin-ts via npm link.

## Steps

### 1. Ensure genshin-ts npm link is registered
- In `D:/MyDrive/Repos/MiliastraWonderland/genshin-ts`, run `npm link`
- It may already be registered — verify first with `npm ls -g --link`

### 2. Set up gsts-sandbox
- Location: `D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox`
- Initialize from the template at `D:/MyDrive/Repos/MiliastraWonderland/genshin-ts/create-genshin-ts/templates/start/`
  - Copy template files into gsts-sandbox (create directory if needed)
- Run `npm install`
- Run `npm link genshin-ts`
- Verify `npm run build` works
- Keep the default template entry as-is (this is a test/validation project)

### 3. Set up elementalist
- Location: `D:/MyDrive/Repos/MiliastraWonderland/elementalist` (folder already exists, empty)
- Initialize from the same template
- Run `npm install`
- Run `npm link genshin-ts`
- Verify `npm run build` works
- This is the production project — set up clean but do not add game logic

### 4. Notes
- The template may use `gsts.config.ts` for configuration — do not modify it beyond what's needed for build
- UID/map separation will be handled separately in the docs directive

## Output
Write a completion report to your `workspace/result.md` with:
- What was done for each project
- Build verification results
- Any issues encountered
