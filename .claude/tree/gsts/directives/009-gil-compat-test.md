# Directive #009
- Domain: injector/compatibility
- Summary: Test GIL injection compatibility after game version update
- Created: 2026-04-09
---

## Context

Game version updated and GIL file format may have changed. Need to verify if existing injection pipeline still works before deeper investigation.

## GIL Files

- Previous version: `C:/Users/Rterg/AppData/LocalLow/miHoYo/Genshin Impact/BeyondLocal/804101570/Beyond_Local_Save_Level/temp.gil`
- New version: `C:/Users/Rterg/AppData/LocalLow/miHoYo/Genshin Impact/BeyondLocal/804101570/Beyond_Local_Save_Level/1073741967.gil`

## Task

1. Read elementalist project's `src/main` directory to understand the source files and config (at `D:/MyDrive/Repos/MiliastraWonderland/elementalist/`)
2. Copy/replicate the relevant source files and gsts config into gsts-sandbox (`D:/MyDrive/Repos/MiliastraWonderland/gsts-sandbox/`)
3. Adjust IDs to match the new GIL file (`1073741967.gil`)
4. Attempt injection using gsts-sandbox's build pipeline
5. Report results: success/failure, any error messages, observations about format differences

## References

- gsts-sandbox and elementalist are both npm-linked to the local genshin-ts fork
- See `notes/protocol/` for GIA/GIL protocol documentation
- The injector module in genshin-ts handles GIL read/write/inject
