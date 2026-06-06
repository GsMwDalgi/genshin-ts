# Node Catalog — Core

All methods are called on `f` (execution flow) inside `g.server().on(...)`.
Exec nodes run in sequence. Data nodes return values (no exec chain needed).

---

## 1. Variables

### Graph Variables
| Method | Type | Description |
|--------|------|-------------|
| `f.get(name)` | data | read graph variable (declared in `g.server({ variables })`) |
| `f.set(name, value)` | exec | write graph variable |

### Custom Variables (per-entity key-value)
| Method | Type | Description |
|--------|------|-------------|
| `f.getCustomVariable(entity, name)` | data | read entity's custom variable |
| `f.queryCustomVariableSnapshot(snapshot, name)` | data | read from snapshot (generic return) |
| `f.setCustomVariable(entity, name, value)` | exec | write entity's custom variable |

---

## 2. List Operations


All list methods support 10 element types: `float`, `int`, `bool`, `config_id`, `entity`, `faction`, `guid`, `prefab_id`, `str`, `vec3`.

### Mutating (exec)
| Method | Description |
|--------|-------------|
| `f.clearList(list)` | clear all |
| `f.concatenateList(target, input)` | append list to list |
| `f.insertValueIntoList(list, index, value)` | insert at index |
| `f.listSorting(list, sortOrder)` | sort |
| `f.modifyValueInList(list, index, value)` | replace at index |
| `f.removeValueFromList(list, index)` | remove at index |

### Query (data)
| Method | Returns | Description |
|--------|---------|-------------|
| `f.getCorrespondingValueFromList(list, index)` | element | get by index |
| `f.getListLength(list)` | int | length |
| `f.getMaximumValueFromList(list)` | element | max |
| `f.getMinimumValueFromList(list)` | element | min |
| `f.listIncludesThisValue(list, value)` | bool | contains check |
| `f.searchListAndReturnValueId(list, value)` | int | index of value |

---

## 3. Dictionary Operations

### Mutating (exec)
| Method | Description |
|--------|-------------|
| `f.clearDictionary(dict)` | clear all |
| `f.removeKeyValuePairsFromDictionaryByKey(dict, key)` | remove by key |
| `f.setOrAddKeyValuePairsToDictionary(dict, key, value)` | set/add entry |
| `f.sortDictionaryByKey(dict, order)` | sort by key → returns `{keyList, valueList}` (dict must be `dict<int, V>`) |
| `f.sortDictionaryByValue(dict, order)` | sort by value → returns `{keyList, valueList}` (dict must be `dict<int, V>`) |

### Query (data)
| Method | Returns | Description |
|--------|---------|-------------|
| `f.createDictionary(keyList, valueList)` | dict | create from two lists |
| `f.getListOfKeysFromDictionary(dict)` | list | all keys |
| `f.getListOfValuesFromDictionary(dict)` | list | all values |
| `f.queryDictionarySLength(dict)` | int | size |
| `f.queryDictionaryValueByKey(dict, key)` | value | lookup by key |
| `f.queryIfDictionaryContainsSpecificKey(dict, key)` | bool | key exists? |
| `f.queryIfDictionaryContainsSpecificValue(dict, value)` | bool | value exists? |

---

## 4. Entity Management

### Actions (exec)
| Method | Description |
|--------|-------------|
| `f.activateDisableModelDisplay(entity, on)` | show/hide model |
| `f.createEntity(guid, unitTagIndexList)` | create entity (tags are int indexes) |
| `f.createPrefab(prefabId, pos, rot, owner, overwriteLvl, lvl, tagList)` | spawn prefab (7 params) |
| `f.createPrefabGroup(prefabGroupId, pos, rot, owner, lvl, tagList, overwriteLvl)` | spawn prefab group (7 params, returns entity_list) |
| `f.destroyEntity(entity)` | destroy |
| `f.modifyEntityFaction(entity, faction)` | change faction (beyond) |
| `f.removeEntity(entity)` | remove (no destroy event) |
| `f.settleStage()` | settle/end the stage (0 params) |
| `f.teleportPlayer(player, position, rotation)` | teleport (3 params) |
| `f.toggleEntityLightSource(entity, lightId, activate)` | toggle light source (3 params) |

---

## 5. Entity Queries

| Method | Returns | Description |
|--------|---------|-------------|
| `f.checkEntitySElementalEffectStatus(entity)` | booleans | affected by each element? |
| `f.getAllEntitiesOnTheField()` | entity_list | all entities |
| `f.getCharacterAttribute(entity)` | attributes | HP, ATK, DEF, etc. (character) |
| `f.getCreationAttribute(entity)` | attributes | HP, ATK, etc. (creation/object) |
| `f.getCreationSCurrentTarget(entity)` | entity | creation's current target |
| `f.getEntitiesWithSpecifiedPrefabOnTheField(prefabId)` | entity_list | entities by prefab |
| `f.getEntityAdvancedAttribute(entity)` | attributes | critRate, critDmg, healBonus, etc. |
| `f.getEntityElementalAttribute(entity)` | attributes | elemental dmgBonus + res per element |
| `f.getEntityForwardVector(entity)` | vec3 | forward direction |
| `f.getEntityListBySpecifiedFaction(list, faction)` | entity_list | filter by faction |
| `f.getEntityListBySpecifiedPrefabId(list, prefabId)` | entity_list | filter by prefab |
| `f.getEntityListBySpecifiedRange(entityList, centerPoint, radius)` | entity_list | filter by range (first param is entity_list) |
| `f.getEntityListBySpecifiedType(list, type)` | entity_list | filter by type |
| `f.getEntityLocationAndRotation(entity)` | vec3, vec3 | position, rotation |
| `f.getEntityRightVector(entity)` | vec3 | right direction |
| `f.getEntityType(entity)` | enum | entity type |
| `f.getEntityUpwardVector(entity)` | vec3 | up direction |
| `f.getListOfEntitiesOwnedByTheEntity(entity)` | entity_list | owned entities |
| `f.getObjectAttribute(entity)` | attributes | HP, ATK, DEF (object) |
| `f.getOwnerEntity(entity)` | entity | owner |
| `f.getPlayerEntityToWhichTheCharacterBelongs(character)` | player | character→player |
| `f.getSelfEntity()` | entity | current graph entity |
| `f.getSpecifiedTypeOfEntitiesOnTheField(entityType)` | entity_list | entities by type |
| `f.queryEntityByGuid(guid)` | entity | guid→entity |
| `f.queryEntityFaction(entity)` | faction | faction |
| `f.queryGuidByEntity(entity)` | guid | entity→guid |
| `f.queryIfEntityIsOnTheField(entity)` | bool | exists? |
| `f.queryIfFactionIsHostile(f1, f2)` | bool | hostile? |

---

## 6. Named Timers


> Prefer `setTimeout`/`setInterval` syntax. Use named timers only when you need pause/resume control.

### Local Timers
| Method | Type | Description |
|--------|------|-------------|
| `f.pauseTimer(entity, name)` | exec | pause local timer |
| `f.resumeTimer(entity, name)` | exec | resume local timer |
| `f.startTimer(entity, name, loop, timerSequence)` | exec | start local timer (4 params, timerSequence: float_list) |
| `f.stopTimer(entity, name)` | exec | stop local timer |

### Global Timers
| Method | Type | Description |
|--------|------|-------------|
| `f.getCurrentGlobalTimerTime(entity, name)` | data | get elapsed time |
| `f.modifyGlobalTimer(entity, name, change)` | exec | modify time (change: float) |
| `f.pauseGlobalTimer(entity, name)` | exec | pause |
| `f.recoverGlobalTimer(entity, name)` | exec | resume |
| `f.startGlobalTimer(entity, name)` | exec | start |
| `f.stopGlobalTimer(entity, name)` | exec | stop |

---

## 7. Signals

| Method | Type | Description |
|--------|------|-------------|
| `f.sendSignal(name, args?)` | exec | send signal with optional typed args |

See guide for `onSignal` receive pattern and 18 supported arg types.

---

## 8. Status & Preset Status

### Preset Status
| Method | Type | Description |
|--------|------|-------------|
| `f.getPresetStatus(entity, id)` | data | get |
| `f.getThePresetStatusValueOfTheComplexCreation(entity, idx)` | data | get (complex creation) |
| `f.setPresetStatus(entity, id, value)` | exec | set |
| `f.setThePresetStatusValueOfTheComplexCreation(entity, idx, value)` | exec | set (complex creation) |

### Unit Status
| Method | Type | Description |
|--------|------|-------------|
| `f.addUnitStatus(applier, target, configId, stacks, paramDict)` | exec+data | apply status (5 params, returns `{applicationResult, slotId}`) |
| `f.listOfSlotIdsQueryingUnitStatus(target, configId)` | data | all slot IDs for status |
| `f.queryIfEntityHasUnitStatus(target, configId)` | data | has status? (configId is config_id) |
| `f.queryUnitStatusApplierBySlotId(target, configId, slotId)` | data | who applied this status? |
| `f.queryUnitStatusStacksBySlotId(target, configId, slotId)` | data | stack count |
| `f.removeUnitStatus(target, configId, removalMethod, remover)` | exec | remove (4 params) |

---

## 9. Collision & Physics

| Method | Type | Description |
|--------|------|-------------|
| `f.activateDisableCollisionTrigger(entity, triggerId, activate)` | exec | toggle trigger (3 params) |
| `f.activateDisableCollisionTriggerSource(entity, activate)` | exec | toggle trigger source |
| `f.activateDisableExtraCollision(entity, collisionId, activate)` | exec | toggle extra collision |
| `f.activateDisableExtraCollisionClimbability(entity, collisionId, activate)` | exec | toggle extra climbability |
| `f.activateDisableNativeCollision(entity, on)` | exec | toggle collision |
| `f.activateDisableNativeCollisionClimbability(entity, on)` | exec | toggle native climbability |
| `f.activateDisablePathfindingObstacle(entity, obstacleId, activate)` | exec | toggle pathfinding (3 params) |
| `f.activateDisablePathfindingObstacleFeature(entity, activate)` | exec | toggle pathfinding feature |
| `f.getAllEntitiesWithinTheCollisionTrigger(entity, triggerId)` | data | entities in trigger (2 params) |

---

## 10. Math & Vector

| Method | Type | Description |
|--------|------|-------------|
| `f.distanceBetweenTwoCoordinatePoints(a, b)` | data | distance between two vec3 → number |
| `f.getRandomFloatingPointNumber(lo, hi)` | data | random float in range → number |
| `f.getRandomInteger(lo, hi)` | data | random int in range → int |
| `f.weightedRandom(weightList)` | data | weighted random pick (int_list) → int |

---

## 11. Print / Debug

| Method | Type | Description |
|--------|------|-------------|
| `f.printString(str)` | exec | debug log |
| `f.forwardingEvent(entity)` | exec | re-fire event |

---

## 12. Utilities

| Method | Type | Description |
|--------|------|-------------|
| `f.queryGameModeAndPlayerNumber()` | data | game mode + player count |
