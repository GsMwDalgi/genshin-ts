# Directive #001
- Domain: tooling/skill
- Summary: Fix node-graph-coder skill — migrate prompt-reference docs into skill folder and update paths
- Created: 2026-03-31
---

## Goal

The `node-graph-coder` skill at `.claude/skills/node-graph-coder/SKILL.md` references `docs/prompt-reference/` which does not exist in the project. The actual files are at an external location: `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\prompt-reference\`.

## Requirements

1. **Copy** the entire `prompt-reference/` directory from `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts-run\docs\prompt-reference\` into `.claude/skills/node-graph-coder/prompt-reference/`

2. **Update SKILL.md** — change all `docs/prompt-reference/` references to use paths relative to the skill folder. The skill runs from the project root, so paths should be `.claude/skills/node-graph-coder/prompt-reference/...`

3. **Scan for any other hardcoded absolute paths** in the copied files and in SKILL.md. Fix any that reference `genshin-ts-run` or other external absolute paths.

4. **Verify** that all referenced files exist at their new paths after migration.

5. **Do NOT modify** the source files in genshin-ts-run — only copy and fix in this project.

## Acceptance Criteria

- All prompt-reference files exist inside `.claude/skills/node-graph-coder/prompt-reference/`
- SKILL.md paths all resolve correctly
- No remaining references to `genshin-ts-run` or external absolute paths
- Skill is self-contained within `.claude/skills/node-graph-coder/`
