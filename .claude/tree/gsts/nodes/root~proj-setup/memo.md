# Memo

## Task
- TASK_ASSIGN — Set up gsts-sandbox and elementalist projects with npm link (received 2026-03-31, COMPLETED)
- TASK_ASSIGN — Write team setup guide (Directive #003) (received 2026-03-31, COMPLETED)
- TASK_ASSIGN — Revise team setup guide for end-user audience (Directive #005) (received 2026-03-31, COMPLETED)
- TASK_ASSIGN — Update package.json + revise setup guide (Directive #006) (received 2026-03-31, COMPLETED)

## Refs
- .claude/tree/gsts/nodes/root/workspace/task-project-setup.md | original setup task spec
- .claude/tree/gsts/nodes/root~proj-setup/workspace/result.md | setup completion report
- .claude/tree/gsts/nodes/root/workspace/task-team-docs.md | docs task spec
- .claude/tree/gsts/directives/003-team-docs.md | directive detail
- notes/team-setup-guide.md | guide (final version)
- .claude/tree/gsts/nodes/root~proj-setup/workspace/result-docs.md | docs completion report
- .claude/tree/gsts/nodes/root/workspace/task-docs-revision.md | revision task spec
- .claude/tree/gsts/directives/005-docs-revision.md | revision directive
- .claude/tree/gsts/nodes/root/workspace/task-pkg-and-docs.md | pkg+docs task spec
- .claude/tree/gsts/directives/006-pkg-and-docs-update.md | pkg+docs directive

## Notes
- genshin-ts globally linked (genshin-ts@0.1.7)
- gsts-sandbox and elementalist: created, installed, linked, build OK
- Both package.json now use "github:GsMwDalgi/genshin-ts#master" (changed from "latest")
- npm install succeeded in both after change (195 packages each)
- Setup guide: general user doc, no "팀원" framing, flow = clone → install → configure → run
- Per-user inject fields: playerId (UID), mapId, gameRegion
- gsts.config.ts should be gitignored (documented as mandatory)
