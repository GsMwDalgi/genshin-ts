# Task: Fix node-graph-coder skill — migrate prompt-reference docs

## Context
The `node-graph-coder` skill at `.claude/skills/node-graph-coder/SKILL.md` references `docs/prompt-reference/` which does not exist in this project. The actual files are at `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\prompt-reference\`.

## Steps

1. **Copy** the entire `prompt-reference/` directory from `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\prompt-reference\` into `.claude/skills/node-graph-coder/prompt-reference/`

2. **Update SKILL.md** — change all `docs/prompt-reference/` references to `.claude/skills/node-graph-coder/prompt-reference/...` (paths relative to project root)

3. **Scan** all copied files AND SKILL.md for any hardcoded absolute paths referencing `genshin-ts-run` or other external locations. Fix them.

4. **Verify** every file path referenced in SKILL.md exists at the new location.

5. **Do NOT modify** any files in genshin-ts-run — only copy and fix within this project.

## Acceptance Criteria
- All prompt-reference files exist inside `.claude/skills/node-graph-coder/prompt-reference/`
- SKILL.md paths all resolve correctly
- No remaining references to `genshin-ts-run` or external absolute paths in any skill files
- Skill is self-contained within `.claude/skills/node-graph-coder/`

## Output
Write a brief completion report to your `workspace/result.md` listing what was copied, what paths were changed, and confirmation of verification.
