# Memo

## Task
- TASK_ASSIGN — Set up gsts-sandbox and elementalist projects (received 2026-03-31, completed but reassigned to root~proj-setup)
- TASK_ASSIGN — Fix parseValue and str() type conversion bugs, Directive #007 (received 2026-03-31, completed)

## Refs
- ../root/workspace/task-project-setup.md | task spec (reassigned)
- ../root/workspace/task-parseval-bugs.md | task spec for parseValue bugs
- .claude/tree/gsts/directives/007-fix-parseval-bugs.md | directive detail

## Notes
- Work was already done for project setup before reassignment; root~proj-setup now owns it
- Bug 1 root cause: onSignal didn't accept signal arg defs, so custom signal params were silently ignored
- Bug 2 root cause: entity case in parseValue had no bigint fallback unlike guid/prefab_id/config_id
- GIA transform already had SIGNAL_ARG_TYPE_MAP and buildSignalNode for monitor_signal, just needed output pin creation
- gsts-sandbox config left at entries: ['./src/SampleForEffect/'] after testing
