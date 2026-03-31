# Task: Update package.json + revise setup guide (Directive #006)

## Directive detail
See `.claude/tree/gsts/directives/006-pkg-and-docs-update.md`

## Part 1: package.json update

In both projects, change the genshin-ts dependency from npm registry to the fork's git URL:

- `D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox/package.json`
- `D:/MyDrive/Repos/MiliastraWonderland/elementalist/package.json`

Change `"genshin-ts": "latest"` (or whatever the current value is) to:
```json
"genshin-ts": "github:GsMwDalgi/genshin-ts#master"
```

After changing, run `npm install` in each project to verify the git URL resolves correctly.

## Part 2: Revise notes/team-setup-guide.md

The guide should read as a **general user setup guide**, not a "teammate guide":

1. Remove any "팀원" framing — write as if for any user
2. Remove genshin-ts clone/npm link steps — users just clone the project and `npm install` (genshin-ts installs from GitHub automatically)
3. Simplify flow to: clone project -> `npm install` -> configure `gsts.config.ts` (UID, map, gameRegion) -> `npm run dev` / `npm run build`
4. Keep existing good content: UID/map confusion prevention, gameRegion explanation, .gitignore guidance, Ctrl+C hint, URL links
5. Omit npm link workflow entirely — not relevant to this audience

## Acceptance Criteria
- Both package.json files reference `github:GsMwDalgi/genshin-ts#master`
- `npm install` succeeds in both projects
- Setup guide reads as clean user-facing document with no "teammate" language
- Guide flow: clone -> install -> configure -> run

## Output
Write completion report to your `workspace/result-pkg-docs.md`.
