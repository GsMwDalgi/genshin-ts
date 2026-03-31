# genshin-ts Coding Guide

You write TypeScript that compiles to node graphs for Genshin Impact UGC (Beyond Mode editor).

---

## Scope Rules (Critical)

**Top-level scope** (compile-time): Free JS/TS. File I/O, npm libs, precompute OK.
Do NOT call `g.server()` or runtime APIs here.

**Graph scope** (`g.server().on(...)` / `gstsServer*`): Only a supported TS subset.
Compiled to node graph — no native JS objects, no async, no recursion.

---

## Entry Pattern

```ts
import { g } from 'genshin-ts/runtime/core'

g.server({
  id: 1073741825,         // target NodeGraph ID (must exist in map)
  name: 'my_graph',       // display name (auto-prefixed _GSTS_)
  mode: 'beyond',         // 'beyond' (default) or 'classic'
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

Chain multiple events: `.on(...).on(...).onSignal(...)`
Same `id` entries merge automatically.

---

## Type System

| TS Type | Graph Type | Usage |
|---------|-----------|-------|
| `number` | float | `3.14` |
| `bigint` | int | `42n` |
| `boolean` | bool | `true` / `false` |
| `string` | str | `'hello'` |

**Rules:**
- Conditions MUST be `boolean`. Use `bool(...)` to convert if needed.
- Use `bigint` for integer ops (modulo, bitwise).
- Lists/dicts must be homogeneous. Mixed types fail.
- Empty arrays need explicit typing: `list('int', [])`.
- `dict(...)` is read-only. Use graph variables for mutable dicts.
- No `Promise`/`async`/`await`/recursion.
- `while(true)` is capped. Use timers or counters.
- `Object.*` / `JSON.*` unsupported in graph scope. Precompute or use `raw(...)`.

---

## Global Functions (Graph Scope)

### Type Helpers
```ts
int(x)      float(x)     str(x)       bool(x)
vec3([x,y,z]) guid(x)     configId(x)  prefabId(x)
entity(x)   faction(x)   idx(x)       // bigint index helper
entity(0)   // placeholder for empty entity params (e.g., ownerEntity in createPrefab)
```

### Collections
```ts
list('int', [1n, 2n, 3n])     // typed list
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

## Graph Variables

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

All list types support autowrap (plain array syntax) for **non-empty arrays**. `list()` wrapper is optional for non-empty arrays. Empty arrays MUST use `list('type', [])` — the compiler cannot infer the type from `[]`.

### Cross-File Data Import

Data can be defined in a separate file and imported into graph variables:

```ts
// src/db.ts — shared data (top-level scope, free JS)
export const INTS = [1n, 2n, 3n, 4n, 5n]
export const PREFABS = [prefabId(200n), prefabId(201n), prefabId(202n)]
export const POSITIONS = [vec3([0, 0, 0]), vec3([10, 0, 5]), vec3([20, 0, -5])]
export const PRICES = { sword: 100n, shield: 200n, potion: 50n }

// src/MyGraph.ts — import and use
import { PREFABS, POSITIONS, PRICES } from './db.js'

g.server({
  id: 1073741826,
  variables: { enemies: PREFABS, spawns: POSITIONS, itemPrices: PRICES }
})
```

---

## Reusable Functions (gstsServer)

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

---

## Signals

### Send
```ts
f.sendSignal('MySignal', [
  { name: 'count', type: 'int', value: 5n },
  { name: 'msg', type: 'str', value: 'hello' },
  { name: 'target', type: 'entity', value: evt.eventSourceEntity },
  { name: 'ids', type: 'int_list', value: [1n, 2n, 3n] }  // raw array OK
])
```

### Receive
```ts
.onSignal('MySignal', (evt, f) => {
  const count = evt.count       // custom args via evt.argName
  const msg = evt.msg
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

## Timers

```ts
// Graph scope — compiled to timer nodes
setTimeout(() => {
  console.log('delayed')
}, 3000)                           // milliseconds

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

## Control Flow

```ts
// if/else
if (bool(condition)) {
  // true branch
} else {
  // false branch
}

// for loop (finite)
for (let i = 0n; i < 10n; i = i + 1n) {
  // loop body
}

// list iteration
for (const item of myList) {
  // item access
}

// switch
switch (value) {
  case 1n: /* ... */ break
  case 2n: /* ... */ break
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `if (x)` where x is not boolean | `if (bool(x))` |
| `let x = 42` (float) for integer ops | `let x = 42n` (bigint) |
| `console.log(a, b)` multiple args | `console.log(a)` single arg only |
| `[]` empty array type ambiguity | `list('int', [])` — non-empty arrays like `[1n, 2n]` autowrap fine |
| `f.get('v') + 1` mixing number/bigint | Match types: `f.get('v') + 1n` |
| `vec3(0, 0, 0)` wrong constructor | `vec3([0, 0, 0])` — takes array |
| `prefabId(200)` without bigint | `prefabId(200n)` — requires bigint |
| `i++` on bigint | `i = i + 1n` — `++` unsupported on bigint |
| `"a" + str(b)` string concatenation | Split into separate `console.log()` calls — `+` compiles to numeric addition |
| `str(prefabIdValue)` | Not supported — `str()` only supports: str, bool, int, float, guid, entity, faction, vec3. NOT: prefab_id, config_id, enum, struct, list, dict |
| `setPlayerSettlementSuccessStatus(p, true)` | Use enum: `f.setPlayerSettlementSuccessStatus(p, SettlementStatus.Victory)`. Requires `import { SettlementStatus } from 'genshin-ts/definitions/enum'`. Values: `.Undefined`, `.Victory`, `.Defeat` |
| `async function` in graph scope | Not supported. Use timers. |
| `JSON.parse()` in graph scope | Move to top-level precompute or `raw(...)` |
