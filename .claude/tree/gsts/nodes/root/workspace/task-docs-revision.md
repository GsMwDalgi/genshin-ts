# Task: Revise team-setup-guide.md for end-user audience (Directive #005)

## Context
The current `notes/team-setup-guide.md` was written targeting a toolchain developer. The actual audience is a teammate who only uses gsts — they don't develop or modify genshin-ts itself.

## Directive detail
See `.claude/tree/gsts/directives/005-docs-revision.md`

## Steps

1. **Read** the current `notes/team-setup-guide.md`

2. **Revise** it to target a gsts end-user:
   - Remove any toolchain development content (forking, building genshin-ts, contributing, npm link internals)
   - Focus on: clone project -> install -> configure personal settings (UID, map) -> build -> dev workflow
   - Keep it simple and actionable — the reader is not deeply technical
   - Language: Korean, technical terms in English
   - Make UID/map confusion prevention prominent

3. **After completing the revision**, send a STATUS_REPORT directly to leader:
   ```
   SendMessage(to: "team-lead", message: "[STATUS_REPORT] [from:root~proj-setup] [to:leader] Directive #005 revision complete. File: notes/team-setup-guide.md. Ready for readability review.")
   ```
   Leader will run a readability review and may send back feedback for another revision pass.

4. **Then report** TASK_RESULT to root (me) as usual.

## Acceptance Criteria
- Guide is clear enough for a non-developer teammate to follow from scratch
- No toolchain internals exposed
- UID/map confusion prevention is prominent

## Output
Write a brief completion report to your `workspace/result-revision.md`.
