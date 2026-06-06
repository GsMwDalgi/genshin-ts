# Pseudocode Spec Language

Write game logic specs using this pseudocode format.
The syntax is Python-like with custom keywords for game concepts.

---

## File Structure

```
graph "GraphName" (id: NUMBER, mode: beyond|classic)

graph_variables:
    NAME: TYPE = INITIAL_VALUE
    ...

on EVENT_NAME(output1, output2, ...):
    LOGIC

on signal "SIGNAL_NAME"(eventSourceEntity, eventSourceGuid, signalSourceEntity, arg1: TYPE, arg2: TYPE, ...):
    LOGIC
```

---

## Graph Declaration

```
graph "DamageTracker" (id: 1073741825, mode: beyond)
```

- `id`: the target graph ID in the map (ask the map creator)
- `mode`: `beyond` (default, fuller features) or `classic`

---

## Types

| Type | Description | Literal Examples |
|------|-------------|-----------------|
| `int` | integer | `0`, `42`, `-1` |
| `float` | decimal | `3.14`, `0.0` |
| `bool` | boolean | `true`, `false` |
| `str` | string | `"hello"` |
| `vec3` | 3D vector | `(1.0, 2.0, 3.0)` |
| `entity` | entity reference | (no literal — always from event/query) |
| `guid` | unique ID | `999` |
| `config_id` | config identifier | `100` |
| `prefab_id` | prefab identifier | `200` |
| `faction` | faction reference | (no literal — from query) |

Array types: append `_list` — e.g., `int_list`, `str_list`, `entity_list`

---

## Variables

### Graph Variables (shared state, persists between events)
```
graph_variables:
    counter: int = 0
    label: str = "ready"
    isActive: bool = false
```

Read/write in any event handler:
```
counter += 1
label = "running"
```

### Custom Variables (per-entity key-value state)
```
setCustomVariable(someEntity, "hp", 100)
let hp: int = getCustomVariable(someEntity, "hp")
```

### Local Variables (temporary, within one handler)
```
let temp: int = damage * 2
let name: str = "player_1"
```

---

## Events

```
on EVENT_NAME(output1, output2, ...):
    LOGIC
```

List only the outputs you use by name. Order does not matter — names are matched, not positions.

```
on attackHits(eventSourceEntity, targetEntity, damage):
    print damage
```

Common events (pseudocode names → TS: add `when` prefix, e.g. `entityCreated` → `whenEntityIsCreated`):
```
on entityCreated(eventSourceEntity, eventSourceGuid):
on entityDestroyed(eventSourceEntity, eventSourceGuid, location, orientation, entityType, faction, damageSource, ownerEntity):
on attackHits(eventSourceEntity, eventSourceGuid, hitTargetEntity, damage, attackTagList, elementalType, elementalAttackPotency):
on attacked(eventSourceEntity, eventSourceGuid, attackerEntity, damage, attackTagList, elementalType, elementalAttackPotency):
on enteringCollisionTrigger(enteringEntity, enteringEntityGuid, triggerEntity, triggerEntityGuid, triggerId):
on exitingCollisionTrigger(exitingEntity, exitingEntityGuid, triggerEntity, triggerEntityGuid, triggerId):
on timerIsTriggered(eventSourceEntity, eventSourceGuid, timerName, timerSequenceId, numberOfLoops):
on theCharacterIsDown(characterEntity, reason, knockdownEntity):
on characterRevives(characterEntity):
on unitStatusChanges(eventSourceEntity, eventSourceGuid, unitStatusConfigId, applierEntity, infiniteDuration, remainingStatusDuration, remainingStatusStacks, originalStatusStacks, slotId):
```

> **Note:** `theCharacterIsDown`, `characterRevives`, collision events do NOT have `eventSourceEntity`/`eventSourceGuid`. They have their own specific output pins as shown above.

---

## Signals

### Send
```
sendSignal("SignalName", arg1: TYPE = VALUE, arg2: TYPE = VALUE, ...)
```

Examples:
```
sendSignal("WaveComplete", waveNum: int = currentWave, score: int = totalScore)
sendSignal("Alert")
sendSignal("SpawnAt", pos: vec3 = spawnPosition, count: int = 5)
```

### Receive
```
on signal "SignalName"(eventSourceEntity, eventSourceGuid, signalSourceEntity, arg1: TYPE, arg2: TYPE, ...):
    LOGIC
```

Example:
```
on signal "WaveComplete"(eventSourceEntity, eventSourceGuid, signalSourceEntity, waveNum: int, score: int):
    print "Wave done"
    print waveNum
    if score > 1000:
        sendSignal("BonusRound")
```

---

## Timers

```
after MILLISECONDS:
    LOGIC

every MILLISECONDS:
    LOGIC
```

Examples:
```
after 3000:
    print "3 seconds passed"
    sendSignal("GameStart")

every 1000:
    counter += 1
    print counter
```

---

## Control Flow

### If / Else
```
if condition:
    LOGIC
elif other_condition:
    LOGIC
else:
    LOGIC
```

### For Loop (count)
```
for i in range(10):
    print i
```

### For Loop (list)
```
for entity in entityList:
    destroyEntity(entity)
```

### While (use sparingly — has iteration cap)
```
while counter < 10:
    counter += 1
```

### Guard Pattern (Early Skip)
```
if not condition:
    # wrap remaining logic — do NOT use `return`
    pass
```

Use `if not condition:` to guard the rest of the handler. Do not use `return` — it is not part of this pseudocode language and complicates spec-compiler translation.

---

## Function Calls

Call any function from the function declarations file:
```
destroyEntity(targetEntity)
recoverHp(player(1), 50.0)
teleportPlayer(player(1), (100.0, 50.0, 200.0), (0.0, 0.0, 0.0))

let pos, rot = getEntityLocationAndRotation(self)
let isAlive: bool = queryIfEntityIsOnTheField(targetEntity)
let allPlayers: entity_list = getListOfPlayerEntitiesOnTheField()
```

### Built-in Shorthands

These are pseudocode shorthands — not from the function declarations file:
```
print "hello"                    # shorthand for printString("hello")
print value                      # shorthand for printString(str(value))
```

**Note:** String concatenation with `+` is NOT supported. Use separate `print` calls:
```
# WRONG: print "score: " + str(score)
# RIGHT:
print "score:"
print score
```

---

## Entity References

```
self                  # this logic's own entity
player(1)             # player 1 (1-indexed)
player(2)             # player 2
stage                 # the game stage entity
```

---

## Comments

```
# This is a comment explaining WHY, not what
counter += 1  # allow overlapping rewards
```

Use comments for:
- Design intent that isn't obvious from the code
- Edge cases or special conditions
- NOT for restating what the code does

---

## Complete Example

```
graph "ArenaWaves" (id: 1073741825, mode: beyond)

graph_variables:
    currentWave: int = 0
    enemiesAlive: int = 0
    totalScore: int = 0

on entityCreated(eventSourceEntity, eventSourceGuid):
    # start first wave
    currentWave = 1
    enemiesAlive = 3
    print "Arena initialized"
    for i in range(3):
        createPrefab(200, (0.0, 0.0, 0.0), (0.0, 0.0, 0.0))

on entityDestroyed(eventSourceEntity, eventSourceGuid):
    enemiesAlive -= 1
    totalScore += 10

    if enemiesAlive <= 0:
        # wave cleared — bonus points
        totalScore += currentWave * 100

        if currentWave >= 5:
            print "All waves complete!"
            print totalScore
            settleStage()
        else:
            # next wave after delay
            after 3000:
                currentWave += 1
                enemiesAlive = currentWave * 3
                for i in range(enemiesAlive):
                    createPrefab(200, (0.0, 0.0, 0.0), (0.0, 0.0, 0.0))

on theCharacterIsDown(characterEntity, reason, knockdownEntity):
    # any player death ends the game
    print "Game Over"
    totalScore = 0
    settleStage()
```
