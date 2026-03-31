# Task: Write unified protocol documentation (Directive #008, Phase 2)

## Context
You are writing unified GIA/GIL protocol documentation by merging two source sets. A structure plan has been prepared by the doc-struct node.

## Structure Plan
Read the full plan at: `.claude/tree/gsts/nodes/root~doc-struct/workspace/structure-plan.md`

This plan contains:
- Inventory of both source sets
- Overlap/conflict map with resolutions
- Proposed unified file layout (18 files)
- Merge notes with specific conflict resolutions
- Language recommendation (bilingual)

## Sources
1. **Primary (authoritative)**: `D:/MyDrive/Repos/MiliastraWonderland/genshin-ts-run/docs/gia-protocol/` — 14 files, Korean
2. **Secondary**: `notes/protocol/` — 5 files, English (these will be replaced by your output)

## Instructions

1. Read the structure plan thoroughly
2. Read all source files from both sets
3. Write the 18 unified files to `notes/protocol/`, replacing the current 5 files
4. Follow the merge strategy in the plan:
   - Primary source is authoritative for binary/protocol facts
   - Preserve confidence levels (CONFIRMED/INFERRED/SPECULATED)
   - Use the specific conflict resolutions noted in the plan
   - Bilingual approach: English for technical/structural content, Korean for explanatory context
5. Delete the 5 old files that are being replaced (gia-structure.md, gil-structure.md, gia-vs-gil.md, graph-encoding.md, injection-flow.md) and write the new ones

## Output
Write completion report to your `workspace/result.md` listing all files created and any issues encountered.
