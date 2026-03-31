# Result: Project Setup

## gsts-sandbox
- Location: `D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox`
- Initialized from `create-genshin-ts/templates/start/` template
- `_gitignore` renamed to `.gitignore`
- `package.json` name set to `gsts-sandbox`
- `npm install` completed (182 packages)
- `npm link genshin-ts` completed (linked to local fork)
- `npm run build` passed — compiled `src/main.ts` to `dist/src/main.gia`

## elementalist
- Location: `D:/MyDrive/Repos/MiliastraWonderland/elementalist`
- Initialized from same template (folder already existed with .git and .vs)
- `_gitignore` renamed to `.gitignore`
- `package.json` name set to `elementalist`
- `npm install` completed (reused cached packages)
- `npm link genshin-ts` completed (linked to local fork)
- `npm run build` passed — compiled `src/main.ts` to `dist/src/main.gia`

## npm link status
- `genshin-ts@0.1.7` was already globally linked from `D:\MyDrive\Repos\MiliastraWonderland\genshin-ts`
- Both projects successfully linked to it

## Issues
- First `npm install` for gsts-sandbox failed with Windows ENOTEMPTY/EBUSY errors (typical Google Drive + npm file locking). Resolved by cleaning node_modules and retrying.
- No issues with elementalist setup.
