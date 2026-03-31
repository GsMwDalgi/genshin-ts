# Directive #005
- Domain: documentation
- Summary: Revise team setup guide for gsts end-user audience, then readability review cycle
- Created: 2026-03-31
---

## Goal

The team setup guide (`notes/team-setup-guide.md`) was written targeting a toolchain developer. The actual audience is a **teammate who only uses gsts** — they don't develop or modify genshin-ts itself. They just need to:
- Install and set up gsts-sandbox or elementalist
- Configure their own UID and map file
- Run build/dev commands
- Not confuse their settings with other users'

## Requirements

1. **Revise** `notes/team-setup-guide.md` to target a gsts end-user:
   - Remove any toolchain development content (forking, building genshin-ts, contributing)
   - Focus on: clone project → install → configure personal settings (UID, map) → build → dev workflow
   - Keep it simple and actionable — the reader is not deeply technical
   - Language: Korean, technical terms in English

2. After revision, **send the file to leader** for readability review. Leader will coordinate a review agent.

## Acceptance Criteria

- Guide is clear enough for a non-developer teammate to follow from scratch
- No toolchain internals exposed
- UID/map confusion prevention is prominent
