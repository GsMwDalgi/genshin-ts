# Task: Write team setup guide (Directive #003)

## Context
Directive #002 (project setup) is complete. Both gsts-sandbox and elementalist are set up with npm link to the local genshin-ts fork. Now write a teammate setup guide documenting how to replicate this setup.

## Reference
- Setup results: `.claude/tree/gsts/nodes/root~dev/workspace/result.md` (what was done for #002)
- Directive detail: `.claude/tree/gsts/directives/003-team-docs.md`

## Steps

1. **Stop any #002 work** — that directive is already complete (root~dev finished it). Do not redo setup.

2. **Read the template and config** to understand the actual structure:
   - `create-genshin-ts/templates/start/` — template files
   - Look for `gsts.config.ts` in the template or in gsts-sandbox/elementalist to understand config structure
   - Check what UID and map file settings exist

3. **Write the guide** at `notes/team-setup-guide.md` (in the genshin-ts project root):
   - Language: Korean, technical terms in English
   - Prerequisites (Node.js, npm, git)
   - How to clone the forked genshin-ts repo
   - How to run `npm install` and `npm link` in genshin-ts
   - How to set up gsts-sandbox or elementalist (clone, npm install, npm link genshin-ts)
   - How to configure their own UID and map file name in `gsts.config.ts`
   - How to run build/dev commands
   - Warning about not mixing up UID/map files between users

4. **UID/map confusion prevention**:
   - Document that each user MUST set their own UID and map file path in `gsts.config.ts`
   - Investigate if `.env` or environment variables are feasible for per-user settings
   - If `.env` is feasible, add `.env` to `.gitignore` in gsts-sandbox and elementalist

5. **Create `notes/` directory** if it doesn't exist

## Output
Write a completion report to your `workspace/result-docs.md` summarizing what was written and any findings about config structure.
