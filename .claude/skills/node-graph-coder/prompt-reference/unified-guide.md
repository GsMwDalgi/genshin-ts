# genshin-ts Unified Guide

TypeScript that compiles to node graphs for Genshin Impact UGC (Beyond World editor).

---

## 1. Project Overview

### What It Does

```
User TS code (.ts) → compiled node calls (.gs.ts) → IR (.json) → injectable binary (.gia)
```

- Write logic in TypeScript inside `g.server().on(...)` handlers
- The compiler translates it into a node graph that runs in the game editor
- Output files: `.gs.ts` (expanded node calls), `.json` (IR/connections), `.gia` (final binary)

### Project Layout

```
src/              — entry TS files (your code goes here)
src/reference/    — example code for reference
dist/             — build outputs (.gs.ts, .json, .gia) — do NOT edit
gsts.config.ts    — compile/inject configuration
```

### Commands

| Command | Description |
|---------|-------------|
| `npm run build` | full compile |
| `npm run dev` | incremental compile (watch mode, auto-inject if configured) |
| `npm run maps` | list recently saved maps |
| `npm run typecheck` | TypeScript type check |
| `npm run lint` | ESLint (includes genshin-ts custom rules) |

### Debugging

When something is wrong, compare intermediate outputs:
1. `.gs.ts` — verify compiled node calls match intent
2. `.json` — verify connections and types
3. `.gia` — final output (binary, not human-readable)

---

## 2. Game Concepts

### Entities

Everything in the game world is an entity:
- **Character**: playable, controlled by a player. Has HP, ATK, DEF, movement speed.
- **Creation / Object**: placed in the map — triggers, devices, monsters, NPCs.
- **Player**: a connected human. Owns characters. Referenced by `player(1)`, `player(2)` (1-indexed).

Every entity has: GUID (unique instance ID), Prefab ID (template type), Config ID, Faction, position/rotation (vec3).

Key references: `self` (this graph's entity), `player(1)`, `stage` / `level` (stage entity).

### Events and Signals

**Events** fire automatically (entity created/destroyed, attacked, collision, timer, etc.) — your logic reacts to them.

**Signals** are messages between logic blocks. Defined by name, can carry typed data. One block sends, another receives. Must be pre-defined in the editor with argument types.

### Variables

- **Graph variables**: shared across the logic block, persist between events. Declared with types and initial values.
- **Custom variables**: per-entity key-value pairs. Each entity can have different values for the same key.
- **Local variables**: temporary, within one event handler only.

### Other Systems

- **Factions**: teams/sides. Can be hostile. Attacks only damage hostile factions by default.
- **Collision triggers**: invisible zones detecting enter/exit.
- **Combat**: attacks, HP, damage, shields, elemental reactions, down/revive.
- **Unit status**: buff/debuff with config ID, stacks, duration.
- **Preset status**: simple integer flags per entity (phase 1/2/3, enabled/disabled).
- **Timers**: delayed or repeated actions (milliseconds). Can pause/resume/stop.
- **Shop/Inventory**: per-player storage, programmable shops.
- **Equipment**: equippable items with affixes (modifiers).
- **Settlement/Ranking**: end-of-game scoring and results.
- **Other**: minimap markers, camera, UI controls, audio, tags, aggro, patrol, achievement, deck selector, chat, environment/time.

---

## 3. Scope Rules (Critical)

**Top-level scope** (compile-time): Free JS/TS. File I/O, npm libs, precompute OK.
Do NOT call `g.server()` or runtime APIs here. Top-level code may run multiple times (incremental builds, multi-entry). Be careful with file I/O or randomness here.

**Graph scope** (`g.server().on(...)` / `gstsServer*`): Only a supported TS subset.
Compiled to node graph — no native JS objects, no async, no recursion.

---

## 4. g.server() Options

```ts
import { g } from 'genshin-ts/runtime/core'

g.server({
  id: 1073741825,         // target NodeGraph ID (must exist in map)
  name: 'my_graph',       // display name (auto-prefixed _GSTS_)
  mode: 'beyond',         // 'beyond' (default) or 'classic'
  type: 'server',         // graph type (default server/entity). 'class' not allowed in classic mode
  prefix: true,           // auto-add _GSTS_ prefix (default true)
  lang: 'zh',             // enable Chinese event names and function aliases (optional)
  variables: {            // graph variables (optional)
    counter: 0n,
    label: 'hello'
  }
}).on('whenEntityIsCreated', (evt, f) => {
  // evt = event output pins
  // f = execution flow functions (actions, queries)
  console.log('entity created')
})
```

- Chain multiple events: `.on(...).on(...).onSignal(...)`
- Same `id` entries merge automatically across files.
- `mode: 'beyond'` (default): fuller node capability. `mode: 'classic'`: narrower, `type: 'class'` not allowed.
- `lang: 'zh'`: enables Chinese event names and function aliases.

### Injection Safety

- The target `id` must exist in the map.
- The target graph must be empty or name starts with `_GSTS`, otherwise injection is blocked.
- Override with `inject.skipSafeCheck = true` in `gsts.config.ts` if you know what you're doing.
- After creating a new graph in the editor, save the map before injecting.

### gsts.config.ts

```ts
import type { GstsConfig } from 'genshin-ts'

const config: GstsConfig = {
  compileRoot: '.',
  entries: ['./src'],          // files or folders to compile
  outDir: './dist',
  inject: {
    gameRegion: 'Global',      // 'Global' or 'China'
    playerId: 1,
    mapId: 1073741849,         // from npm run maps
  }
}

export default config
```

- `entries`: file (`'./src/file.ts'`) or folder (`'./src/folder/'`).
- `npm run maps` lists recent maps to help locate `mapId`.
- Injection creates backups automatically.

---

## 5. Type System

| TS Type | Graph Type | Literal |
|---------|-----------|---------|
| `number` | float | `3.14` |
| `bigint` | int | `42n` |
| `boolean` | bool | `true` / `false` |
| `string` | str | `'hello'` |

**Rules:**
- Conditions MUST be `boolean`. Use `bool(...)` to convert if needed.
- Use `bigint` for integer ops (modulo, bitwise).
- Lists/dicts must be homogeneous. Mixed types fail.
- Empty arrays need explicit typing: `list('int', [])`.
- Non-empty arrays autowrap fine: `[1n, 2n]`, `[prefabId(200n)]`.
- `dict(...)` is read-only. Use graph variables for mutable dicts.
- No `Promise`/`async`/`await`/recursion.
- `while(true)` is capped. Use timers or counters.
- `Object.*` / `JSON.*` unsupported in graph scope. Precompute or use `raw(...)`.
- Use `let` to force a local variable node; `const` may be optimized into direct wiring.

---

## 6. Global Functions (Graph Scope)

### Type Helpers
```ts
int(x)      float(x)     str(x)       bool(x)
vec3([x,y,z]) guid(x)     configId(x)  prefabId(x)
entity(x)   faction(x)   idx(x)       // bigint index helper
entity(0)   // placeholder for empty entity params
```

`str()` supports: str, bool, int, float, guid, entity, faction, vec3.
NOT supported: config_id, prefab_id, enum, struct, list, dict.

### Collections
```ts
list('int', [1n, 2n, 3n])     // typed list (required for empty arrays)
dict({ key: value, ... })     // read-only dictionary
```

### Entities
```ts
player(1)    // player entity (1-indexed)
self         // current graph entity
stage        // stage entity
level        // level entity
```

### Logging
```ts
print(str(...))              // most stable
console.log(x)               // single arg only, auto-rewritten
f.printString(str(value))    // explicit node call
```

### Math
```ts
Math.abs(x)   Math.floor(x)   Math.ceil(x)   Math.round(x)
Math.min(a,b) Math.max(a,b)   Math.sqrt(x)   Math.PI
Math.sin(x)   Math.cos(x)     Math.atan2(y,x)
Random.Range(lo, hi)
Vector3.Distance(a, b)
```

---

## 7. Graph Variables

```ts
g.server({
  id: 1073741825,
  variables: {
    counter: 0n,                                              // int
    multiplier: 1.5,                                          // float
    label: 'hello',                                           // str
    active: true,                                             // bool
    counts: [3n, 5n, 7n],                                     // int_list
    rates: [0.5, 1.0, 1.5],                                   // float_list
    names: ['alpha', 'bravo', 'charlie'],                      // str_list
    flags: [true, false, true],                                // bool_list
    enemies: [prefabId(200n), prefabId(201n)],                 // prefab_id_list
    rewards: [configId(100n), configId(101n)],                 // config_id_list
    targets: [guid(1n), guid(2n)],                             // guid_list
    positions: [vec3([0, 0, 0]), vec3([10, 0, 5])],            // vec3_list
    prices: { sword: 100n, shield: 200n },                     // dict
  }
}).on('whenEntityIsCreated', (_evt, f) => {
  const v = f.get('counter')       // read (typed by declaration)
  f.set('counter', v + 1n)         // write
})
```

All list types support autowrap (plain array syntax) for non-empty arrays. Empty arrays MUST use `list('type', [])`.

### Cross-File Data Import

```ts
// src/db.ts — shared data (top-level scope, free JS)
export const PREFABS = [prefabId(200n), prefabId(201n), prefabId(202n)]
export const PRICES = { sword: 100n, shield: 200n, potion: 50n }

// src/MyGraph.ts — import and use
import { PREFABS, PRICES } from './db.js'

g.server({
  id: 1073741826,
  variables: { enemies: PREFABS, itemPrices: PRICES }
})
```

---

## 8. Signals

### Send
```ts
// without args
f.sendSignal('EnemyKilled')

// with args
f.sendSignal('WaveCleared', [
  { name: 'count', type: 'int', value: 5n },
  { name: 'msg', type: 'str', value: 'hello' },
  { name: 'target', type: 'entity', value: evt.eventSourceEntity },
  { name: 'ids', type: 'int_list', value: [1n, 2n, 3n] }
])
```

### Receive
```ts
// without args
.onSignal('EnemyKilled', (evt, f) => {
  // evt.signalSourceEntity available
})

// with args
.onSignal('WaveCleared', (evt, f) => {
  console.log(evt.count)      // access args via evt.argName
  console.log(evt.msg)
}, [
  { name: 'count', type: 'int' },
  { name: 'msg', type: 'str' },
  { name: 'target', type: 'entity' },
  { name: 'ids', type: 'int_list' }
])
```

Signal types: `entity`, `guid`, `int`, `bool`, `float`, `str`, `vec3`, `config_id`, `prefab_id`
Array types: `guid_list`, `int_list`, `bool_list`, `float_list`, `str_list`, `entity_list`, `vec3_list`, `config_id_list`, `prefab_id_list`

**Note:** `entity` type is connection-only (no literal value). Pass event outputs or variable reads.

---

## 9. Timers

```ts
setTimeout(() => {
  console.log('delayed')
}, 3000)

const id = setInterval(() => {
  console.log('tick')
}, 1000)
clearInterval(id)
```

- Compiler builds timer name pools automatically.
- `setInterval` <= 100ms triggers performance warning.
- Override pool: `// @gsts:timerPool=4`
- Callbacks capture by value. Dict captures unsupported.

---

## 10. Control Flow

```ts
if (bool(condition)) {
  // true branch
} else {
  // false branch
}

for (let i = 0n; i < 10n; i = i + 1n) {
  // loop body (bigint counter)
}

for (const item of myList) {
  // list iteration
}

switch (value) {
  case 1n: /* ... */ break
  case 2n: /* ... */ break
}
```

---

## 11. Reusable Functions (gstsServer)

```ts
function gstsServerSum(a: bigint, b: bigint) {
  const total = a + b
  return total                     // single trailing return only
}

g.server({ id: 1073741825 }).on('whenEntityIsCreated', (_evt, f) => {
  const v = gstsServerSum(1n, 2n)
  f.printString(str(v))
})
```

Rules:
- Must be top-level, prefixed `gstsServer`.
- Params: identifiers only (no destructuring/default/rest).
- Only one trailing `return <expr>`.
- Callable only inside `g.server().on(...)` or another `gstsServer*`.
- Inside `gstsServer*`, use `gsts.f` directly (no need to pass `f`).

---

## 12. Function & Event Lookup

When type hints are not enough, search the source definitions:
- Node functions: `node_modules/genshin-ts/dist/src/definitions/nodes.ts`
- Events: `node_modules/genshin-ts/dist/src/definitions/events.ts`
- Use keywords (event name, function name, Chinese alias) to locate comments and params.
- Chinese and English alias keywords are both supported.

---

## 13. Common Mistakes

| Mistake | Fix |
|---------|-----|
| `if (x)` where x is not boolean | `if (bool(x))` |
| `let x = 42` (float) for integer ops | `let x = 42n` (bigint) |
| `console.log(a, b)` multiple args | `console.log(a)` single arg only |
| `[]` empty array type ambiguity | `list('int', [])` — non-empty `[1n, 2n]` autowrap fine |
| `f.get('v') + 1` mixing number/bigint | Match types: `f.get('v') + 1n` |
| `vec3(0, 0, 0)` wrong constructor | `vec3([0, 0, 0])` — takes array |
| `prefabId(200)` without bigint | `prefabId(200n)` — requires bigint |
| `i++` on bigint | `i = i + 1n` — `++` unsupported on bigint |
| `"a" + str(b)` string concatenation | Split into separate `console.log()` calls — `+` compiles to numeric addition |
| `str(prefabIdValue)` | Not supported — use `str()` only on: str, bool, int, float, guid, entity, faction, vec3 |
| `setPlayerSettlementSuccessStatus(p, true)` | Use enum: `SettlementStatus.Victory`. Requires `import { SettlementStatus } from 'genshin-ts/definitions/enum'` |
| `async function` in graph scope | Not supported. Use timers. |
| `JSON.parse()` in graph scope | Move to top-level precompute or `raw(...)` |
| Store `evt.xxx` in local variable | Use `evt.xxx` directly — local variable adds unnecessary node pairs |
