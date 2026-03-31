# Available Events

These are all events you can listen for in pseudocode specs.
Each event provides named output pins you can reference in the handler body.
Mode: `B` = beyond only, `C` = classic only, blank = both.

Most events provide `eventSourceEntity: entity` and `eventSourceGuid: guid`.
**Exceptions:** Character State events and Collision events have their own output pins instead — see sections 2 and 4.

Signals are handled separately with `on signal "Name"(...)` syntax — see section 8.

---

## 1. Entity Lifecycle

```python
on entityCreated(eventSourceEntity: entity, eventSourceGuid: guid): ...
on entityDestroyed(eventSourceEntity: entity, eventSourceGuid: guid, location: vec3, orientation: vec3, entityType: int, faction: faction, damageSource: entity, ownerEntity: entity): ...
on entityFactionChanges(eventSourceEntity: entity, eventSourceGuid: guid, preChangeFaction: faction, postChangeFaction: faction): ...  # B
on entityIsRemovedDestroyed(eventSourceGuid: guid): ...  # no eventSourceEntity
```

---

## 2. Character State

> These events do NOT have `eventSourceEntity` / `eventSourceGuid`. They have their own specific output pins.

```python
on allPlayerSCharactersAreDown(playerEntity: entity, reason: int): ...
on allPlayerSCharactersAreRevived(playerEntity: entity): ...
on characterMovementSpdMeetsCondition(unitStatusConfigId: config_id, conditionComparisonType: int, conditionComparisonValue: float, currentMovementSpd: float): ...
on characterRevives(characterEntity: entity): ...
on playerIsAbnormallyDownedAndRevives(playerEntity: entity): ...
on playerTeleportCompletes(playerEntity: entity, playerGuid: guid): ...
on theActiveCharacterChanges(playerEntity: entity, playerGuid: guid, previousActiveCharacterEntity: entity, currentActiveCharacterEntity: entity): ...  # C
on theCharacterIsDown(characterEntity: entity, reason: int, knockdownEntity: entity): ...
```

---

## 3. Combat

```python
on attacked(eventSourceEntity: entity, eventSourceGuid: guid, attackerEntity: entity, damage: float, attackTagList: str_list, elementalType: int, elementalAttackPotency: float): ...
on attackHits(eventSourceEntity: entity, eventSourceGuid: guid, hitTargetEntity: entity, damage: float, attackTagList: str_list, elementalType: int, elementalAttackPotency: float): ...
on creationEntersCombat(eventSourceEntity: entity, eventSourceGuid: guid): ...
on creationLeavesCombat(eventSourceEntity: entity, eventSourceGuid: guid): ...
on elementalReactionEventOccurs(eventSourceEntity: entity, eventSourceGuid: guid, elementalReactionType: int, triggererEntity: entity, triggererEntityGuid: guid): ...
on enteringAnInterruptibleState(eventSourceEntity: entity, eventSourceGuid: guid, attacker: entity): ...  # B
on hpIsRecovered(eventSourceEntity: entity, eventSourceGuid: guid, healerEntity: entity, recoveryAmount: float, recoverTagList: str_list): ...
on initiatingHpRecovery(eventSourceEntity: entity, eventSourceGuid: guid, recoverTargetEntity: entity, recoveryAmount: float, recoverTagList: str_list): ...
on selfEntersCombat(eventSourceEntity: entity, eventSourceGuid: guid): ...  # B
on selfLeavesCombat(eventSourceEntity: entity, eventSourceGuid: guid): ...  # B
on shieldIsAttacked(eventSourceEntity: entity, eventSourceGuid: guid, attackerEntity: entity, attackerGuid: guid, unitStatusConfigId: config_id, preAttackLayers: int, postAttackLayers: int, shieldValueOfThisUnitStatusBeforeAttack: float, shieldValueOfThisUnitStatusAfterAttack: float): ...
```

---

## 4. Collision

> These events do NOT have `eventSourceEntity` / `eventSourceGuid`. They have their own specific output pins.

```python
on enteringCollisionTrigger(enteringEntity: entity, enteringEntityGuid: guid, triggerEntity: entity, triggerEntityGuid: guid, triggerId: int): ...
on exitingCollisionTrigger(exitingEntity: entity, exitingEntityGuid: guid, triggerEntity: entity, triggerEntityGuid: guid, triggerId: int): ...
on onHitDetectionIsTriggered(onHitHurtbox: bool, onHitEntity: entity, onHitLocation: vec3): ...
```

---

## 5. Variables

```python
on complexCreationPresetStatusChanges(eventSourceEntity: entity, eventSourceGuid: guid, presetStatusIndex: int, preChangeValue: any, postChangeValue: any): ...
on customVariableChanges(eventSourceEntity: entity, eventSourceGuid: guid, variableName: str, preChangeValue: any, postChangeValue: any): ...
on nodeGraphVariableChanges(eventSourceEntity: entity, eventSourceGuid: guid, variableName: str, preChangeValue: any, postChangeValue: any): ...
on presetStatusChanges(eventSourceEntity: entity, eventSourceGuid: guid, presetStatusId: int, preChangeValue: any, postChangeValue: any): ...
```

---

## 6. Status

```python
on unitStatusChanges(eventSourceEntity: entity, eventSourceGuid: guid, unitStatusConfigId: config_id, applierEntity: entity, infiniteDuration: bool, remainingStatusDuration: float, remainingStatusStacks: int, originalStatusStacks: int, slotId: int): ...
on unitStatusEnds(eventSourceEntity: entity, eventSourceGuid: guid, unitStatusConfigId: config_id, applierEntity: entity, infiniteDuration: bool, remainingStatusDuration: float, remainingStatusStacks: int, removerEntity: entity, removalReason: int, slotId: int): ...
```

---

## 7. Timer

```python
on globalTimerIsTriggered(eventSourceEntity: entity, eventSourceGuid: guid, timerName: str): ...
on timerIsTriggered(eventSourceEntity: entity, eventSourceGuid: guid, timerName: str, timerSequenceId: int, numberOfLoops: int): ...
```

---

## 8. Signal

Signals use a different syntax. See `pseudocode-spec.md` for send/receive patterns.

```python
on signal "SignalName"(eventSourceEntity: entity, eventSourceGuid: guid, signalSourceEntity: entity, arg1: type, arg2: type, ...): ...
```

---

## 9. Motion

```python
on basicMotionDeviceStops(eventSourceEntity: entity, eventSourceGuid: guid, motionDeviceName: str): ...
on creationReachesPatrolWaypoint(creationEntity: entity, creationGuid: guid, currentPatrolTemplateId: int, currentPathIndex: int, currentReachedWaypointId: int, nextWaypointId: int): ...
on pathReachesWaypoint(eventSourceEntity: entity, eventSourceGuid: guid, motionDeviceName: str, pathPointId: int): ...
```

---

## 10. UI

```python
on deckSelectorIsComplete(eventSourceEntity: entity, eventSourceGuid: guid, targetPlayer: entity, selectionResultList: int_list, completionReason: int, deckSelectorIndex: int): ...
on tabIsSelected(eventSourceEntity: entity, eventSourceGuid: guid, tabId: int, selectorEntity: entity): ...
on textBubbleIsCompleted(eventSourceEntity: entity, eventSourceGuid: guid, bubbleOwnerEntity: entity, characterEntity: entity, textBubbleConfigurationId: config_id, textBubbleCompletionCount: int): ...
on uiControlGroupIsTriggered(eventSourceEntity: entity, eventSourceGuid: guid, uiControlGroupCompositeIndex: int, uiControlGroupIndex: int): ...
```

---

## 11. Class & Skill (Beyond)

```python
on aggroTargetChanges(eventSourceEntity: entity, eventSourceGuid: guid, preChangeAggroTarget: entity, postChangeAggroTarget: entity): ...  # B
on playerClassChanges(eventSourceEntity: entity, eventSourceGuid: guid, preModificationClassConfigId: config_id, postModificationConfigId: config_id): ...  # B
on playerClassIsRemoved(eventSourceEntity: entity, eventSourceGuid: guid, preModificationClassConfigId: config_id, postModificationConfigId: config_id): ...  # B
on playerClassLevelChanges(eventSourceEntity: entity, eventSourceGuid: guid, preChangeLevel: int, postChangeLevel: int): ...  # B
on skillNodeIsCalled(eventSourceEntity: entity, eventSourceGuid: guid, callerEntity: entity, callerGuid: guid, parameter1: str, parameter2: str, parameter3: str): ...  # B
```

---

## 12. Shop & Inventory

```python
on customShopItemIsSold(eventSourceEntity: entity, eventSourceGuid: guid, shopOwner: entity, shopOwnerGuid: guid, buyerEntity: entity, shopId: int, shopItemId: int, purchaseQuantity: int): ...
on itemIsAddedToInventory(eventSourceEntity: entity, eventSourceGuid: guid, itemOwnerEntity: entity, itemOwnerGuid: guid, itemConfigId: config_id, quantityObtained: int): ...
on itemIsLostFromInventory(eventSourceEntity: entity, eventSourceGuid: guid, itemOwnerEntity: entity, itemOwnerGuid: guid, itemConfigId: config_id, quantityLost: int): ...
on itemsInTheInventoryAreUsed(eventSourceEntity: entity, eventSourceGuid: guid, itemOwnerEntity: entity, itemOwnerGuid: guid, itemConfigId: config_id, amountToUse: int): ...
on sellingInventoryItemsInTheShop(eventSourceEntity: entity, eventSourceGuid: guid, shopOwner: entity, shopOwnerGuid: guid, buyerEntity: entity, shopId: int, itemConfigId: config_id, purchaseQuantity: int): ...
on sellingItemsToTheShop(eventSourceEntity: entity, eventSourceGuid: guid, shopOwner: entity, shopOwnerGuid: guid, sellerEntity: entity, shopId: int, purchaseItemDictionary: dict): ...
on theQuantityOfInventoryCurrencyChanges(eventSourceEntity: entity, eventSourceGuid: guid, currencyOwnerEntity: entity, currencyOwnerGuid: guid, currencyConfigId: config_id, currencyChangeValue: int): ...
on theQuantityOfInventoryItemChanges(eventSourceEntity: entity, eventSourceGuid: guid, itemOwnerEntity: entity, itemOwnerGuid: guid, itemConfigId: config_id, preChangeQuantity: int, postChangeQuantity: int, reasonForChange: int): ...
```

---

## 13. Equipment

```python
on equipmentAffixValueChanges(eventSourceEntity: entity, eventSourceGuid: guid, equipmentOwner: entity, equipmentOwnerGuid: guid, equipmentIndex: int, affixId: int, preChangeValue: float, postChangeValue: float): ...
on equipmentIsEquipped(eventSourceEntity: entity, eventSourceGuid: guid, equipmentHolderEntity: entity, equipmentHolderGuid: guid, equipmentIndex: int): ...
on equipmentIsInitialized(eventSourceEntity: entity, eventSourceGuid: guid, equipmentOwner: entity, equipmentOwnerGuid: guid, equipmentIndex: int): ...
on equipmentIsUnequipped(eventSourceEntity: entity, eventSourceGuid: guid, equipmentOwnerEntity: entity, equipmentOwnerGuid: guid, equipmentIndex: int): ...
```
