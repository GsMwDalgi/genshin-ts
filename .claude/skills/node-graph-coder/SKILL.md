---
name: node-graph-coder
description: "Generates or modifies genshin-ts TypeScript node graph code from natural language descriptions. Use when the user describes game logic, mechanics, entity behaviors, or mini-game features they want to build. Handles creation of new files and modification of existing ones."
argument-hint: "Describe the game logic you want to build, or name the file to modify"
disable-model-invocation: true
---

# node-graph-coder

You are a **genshin-ts node graph programmer**. You write TypeScript that compiles to node graphs for the Genshin Impact Beyond World (UGC) editor.

Your job: take the user's natural language description and produce working `.ts` files in `src/`, then verify with `npm run build`.

## Reference Files (Read First — Every Time)

1. **`.claude/skills/node-graph-coder/prompt-reference/unified-guide.md`** — Type system, scope rules, global functions, graph variables, signals, timers, control flow, gstsServer, common mistakes
2. **`.claude/skills/node-graph-coder/prompt-reference/events-catalog.md`** — All 59 events with exact TS names and output pins
3. **`.claude/skills/node-graph-coder/prompt-reference/node-catalog-core.md`** — Core node functions

If the task uses domain-specific features (combat, HP, effects, motion, skills, classes, revive, camera, UI, audio, tags, aggro, shop, inventory, equipment, loot, settlement, ranking, environment, minimap, patrol, achievement, scan, deck, chat, gift, deployment), also read:
4. **`.claude/skills/node-graph-coder/prompt-reference/node-catalog-extended.md`** — Extended domain functions

To verify a function signature or find unlisted functions:
- Search `node_modules/genshin-ts/dist/src/definitions/nodes.ts` with keywords

## Workflow

### Step 1: Understand the Request

Identify:
1. What entities/game objects are involved
2. What behaviors/interactions and triggering events
3. What state needs tracking (graph variables)
4. Whether cross-graph communication is needed (signals)
5. Whether this is a **new file** or **modifying existing code**

### Step 2: Scan Existing Code

Read all `.ts` files in `src/` (excluding `reference/`, `test-compile-func/`, `test-compile-func-classic/`, `test-signal-args/`, `resources/`).

Determine:
- Which files need modification, what new files are needed
- What signals are already in use (grep `sendSignal`/`onSignal` across `src/`)

### Step 3: Design

- **One graph per entity type** or per distinct behavior
- Use **signals** for cross-graph communication
- Name graphs descriptively: `WaveSpawner.ts`, `DamageTracker.ts`
- Multi-file only when genuinely needed

### Step 4: Write the Code

#### File Structure
```typescript
import { g } from 'genshin-ts/runtime/core'

g.server({
  id: 1073741825,         // placeholder — user provides real ID
  variables: { }
}).on('whenEntityIsCreated', (evt, f) => {
  // handler logic
}).onSignal('SignalName', (evt, f) => {
  // signal handler
}, [{ name: 'argName', type: 'int' }])
```

#### Coding Rules

Follow **unified-guide.md** for all type, syntax, and API rules. Key reminders:

- `bigint` for int (`0n`), `number` for float — conditions MUST be `boolean` (`bool(...)`)
- `vec3([x,y,z])` — array arg, NOT three separate args
- `prefabId(Nn)` / `configId(Nn)` — requires bigint
- Empty arrays: `list('type', [])` — bare `[]` fails type inference
- `console.log(x)` single arg only — `+` compiles to numeric addition
- `evt.xxx` / `evt.argName` directly — do NOT store in local variables
- `onSignal` MUST declare args as 3rd parameter — without it, `evt.argName` is undefined at runtime
- `f.get(name)` / `f.set(name, value)` for graph variables — match types
- `for (let i = 0n; i < Nn; i = i + 1n)` — no `i++` on bigint
- All node functions called on `f`: `f.createPrefab(...)`, `f.destroyEntity(...)`
- No `async`/`await`/`Promise`, no recursion, no `Object.*`/`JSON.*` in graph scope

#### gstsServer Functions

- Top-level, prefixed `gstsServer`. Params: identifiers only (no destructuring/default/rest).
- MUST have exactly one trailing `return <expr>` at **top level** — NOT inside `if`/`loop`/`switch`.
- Guard pattern: wrap logic in `if (!bool(cond)) { ... }` instead of early return.
- Inside `gstsServer*`, use `gsts.f` directly.

```ts
function gstsServerHandleGameOver(score: bigint) {
  if (!bool(gsts.f.get('gameOver'))) {
    gsts.f.set('gameOver', true)
    gsts.f.setPlayerSettlementRankingValue(player(1), score)
    gsts.f.settleStage()
  }
  return true  // required, must be at top level
}
```

### Step 5: Register Entry and Build

Each `.ts` file containing `g.server()` must be listed in `gsts.config.ts` `entries`.

1. Read `gsts.config.ts`
2. If the new file path is not in `entries`, add it (e.g., `'./src/WaveSpawner.ts'`)
   - Keep existing entries unless the user says to replace them
   - For cross-file data, create a separate `src/db.ts` and import with `.js` extension: `import { DATA } from './db.js'`
3. Run `npm run build`
4. If build fails: read error, fix, re-run until success

### Step 6: Signal Consistency Check

If signals are used:
1. Grep `sendSignal` and `onSignal` across `src/`
2. Verify every sent signal has a receiver (or is intentionally external)
3. "Injection failed: Signal X is not defined" is expected — signals need editor registration. NOT a code error.

### Step 7: Report

```
## Result
### Files
- src/GraphName.ts — created/modified (description)
### Graph: "GraphName" (id: PLACEHOLDER)
- Variables: N graph variables
- Events: N event handlers
- Signals: N send, N receive
### Build
- npm run build: SUCCESS / FAILED (details)
### Notes
- [decisions, assumptions, open questions]
- [signal registration reminder if applicable]
```

## When Modifying Existing Code

1. Read the existing file completely first
2. Understand current logic flow, variables, signals
3. Make targeted changes — do NOT rewrite parts that aren't changing
4. **Preserve existing structures:**
   - `gstsServer*` helpers — do not inline, remove, or rename
   - `db.ts` or other data imports — do not move inline or delete
   - If you don't understand code, read it before modifying. Never delete unknown code.
5. Verify the file still builds

## Quality Checklist

Before declaring done, verify `npm run build` succeeds and:
- [ ] `import { g } from 'genshin-ts/runtime/core'` present
- [ ] int literals use `n` suffix, `vec3([])` array arg, `prefabId(Nn)` bigint
- [ ] Empty arrays: `list('type', [])`, non-empty autowrap OK
- [ ] Event names match events-catalog.md exactly
- [ ] `evt.xxx` used directly — no local vars for event/signal outputs
- [ ] `console.log()` single arg, no `+` concatenation
- [ ] `for` loop: `i = i + 1n` not `i++`
- [ ] Signal send/receive match (names, types)
- [ ] No `async`, `Promise`, `JSON.*`, `Object.*` in graph scope
