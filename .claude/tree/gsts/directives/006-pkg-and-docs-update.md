# Directive #006
- Domain: tooling/setup + documentation
- Summary: Update package.json to use git URL and revise setup guide as general user docs
- Created: 2026-03-31
---

## Part 1: package.json update

In both projects, change the genshin-ts dependency from npm registry to the fork's git URL:

- `D:\MyDrive\Repos\MiliastraWonderland\gsts-sandbox\package.json`
- `D:\MyDrive\Repos\MiliastraWonderland\elementalist\package.json`

Change:
```json
"genshin-ts": "latest"
```
To:
```json
"genshin-ts": "github:GsMwDalgi/genshin-ts#master"
```

After changing, run `npm install` in each project to verify the git URL resolves correctly.

## Part 2: Revise notes/team-setup-guide.md

The guide should read as a **general user setup guide**, not a "teammate guide":

1. Remove any "팀원" framing — write it as if for any user setting up the project
2. Remove the genshin-ts clone/npm link steps — users just need to clone the project and run `npm install` (genshin-ts installs automatically from GitHub)
3. Simplify the setup flow to:
   - Clone the project (gsts-sandbox or elementalist)
   - `npm install`
   - Configure `gsts.config.ts` (UID, map, gameRegion)
   - `npm run dev` or `npm run build`
4. Keep all the existing good content: UID/map confusion prevention, gameRegion explanation, .gitignore guidance, Ctrl+C hint, URL links
5. The npm link workflow for local development is NOT relevant to this guide's audience — omit it entirely

## Acceptance Criteria

- Both package.json files reference `github:GsMwDalgi/genshin-ts#master`
- `npm install` succeeds in both projects
- Setup guide reads as a clean user-facing document with no "teammate" language
- Guide flow: clone → install → configure → run
