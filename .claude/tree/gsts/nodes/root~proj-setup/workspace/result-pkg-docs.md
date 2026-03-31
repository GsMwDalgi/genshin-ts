# Result: package.json Update + Guide Revision (Directive #006)

## Part 1: package.json Update

Changed `"genshin-ts": "latest"` → `"genshin-ts": "github:GsMwDalgi/genshin-ts#master"` in both:

- `D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox/package.json`
- `D:/MyDrive/Repos/MiliastraWonderland/elementalist/package.json`

`npm install` results:
- gsts-sandbox: added 53 packages, changed 7 packages, audited 195 packages — OK
- elementalist: added 53 packages, changed 1 package, audited 195 packages — OK

Both resolve the git URL correctly.

## Part 2: Guide Revision

Revised `notes/team-setup-guide.md`:

**Changes made:**
- Title changed to "Genshin-TS 환경 설정 가이드" (removed "팀" framing)
- Intro updated: "프로젝트를 처음 설정하는 사람을 위한 가이드" (generic, not teammate-specific)
- Removed "URL은 팀장에게 확인하세요" note from clone step
- No npm link steps present (already removed in prior revision)
- Restored `.gitignore` guidance section (mandatory tone, from feedback pass)
- Restored VS Code recommendation line
- All existing good content preserved: UID/map confusion warning, checklist, gameRegion explanation, Ctrl+C hint, URL links, `npm run maps` order dependency note

**Final flow:** clone → `npm install` → configure `gsts.config.ts` → `npm run build` / `npm run dev`

## Acceptance Criteria Check
- Both package.json reference `github:GsMwDalgi/genshin-ts#master`: yes
- `npm install` succeeds in both: yes
- Guide reads as clean user-facing document with no "teammate" language: yes
- Guide flow: clone → install → configure → run: yes
