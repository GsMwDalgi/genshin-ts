# Events Catalog

Register with `g.server({ id }).on('eventName', (evt, f) => { ... })`.

Most events provide `evt.eventSourceEntity` (entity) and `evt.eventSourceGuid` (guid).
**Exceptions:** Character State events (`whenTheCharacterIsDown`, `whenCharacterRevives`, etc.) and Collision events use their own specific output pins instead — see tables below.
`onSignal` additionally provides `evt.signalSourceEntity`.

Mode: `B` = beyond only, `C` = classic only, blank = both.

---

## 1. Entity Lifecycle

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenEntityFactionChanges` | B | preChangeFaction, postChangeFaction |
| `whenEntityIsCreated` | | (fixed outputs only) |
| `whenEntityIsDestroyed` | | location, orientation, entityType, faction, damageSource, ownerEntity |
| `whenEntityIsRemovedDestroyed` | | eventSourceGuid only (no eventSourceEntity) |

---

## 2. Character State

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenAllPlayerSCharactersAreDown` | | playerEntity, reason |
| `whenAllPlayerSCharactersAreRevived` | | playerEntity |
| `whenCharacterMovementSpdMeetsCondition` | | unitStatusConfigId, conditionComparisonType, conditionComparisonValue, currentMovementSpd |
| `whenCharacterRevives` | | characterEntity |
| `whenPlayerIsAbnormallyDownedAndRevives` | | playerEntity |
| `whenPlayerTeleportCompletes` | | playerEntity, playerGuid |
| `whenTheActiveCharacterChanges` | C | playerEntity, playerGuid, previousActiveCharacterEntity, currentActiveCharacterEntity |
| `whenTheCharacterIsDown` | | characterEntity, reason, knockdownEntity |

---

## 3. Combat

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenAttacked` | | attackerEntity, damage, attackTagList, elementalType, elementalAttackPotency |
| `whenAttackHits` | | hitTargetEntity, damage, attackTagList, elementalType, elementalAttackPotency |
| `whenCreationEntersCombat` | | (fixed outputs only) |
| `whenCreationLeavesCombat` | | (fixed outputs only) |
| `whenElementalReactionEventOccurs` | | elementalReactionType, triggererEntity, triggererEntityGuid |
| `whenEnteringAnInterruptibleState` | B | attacker |
| `whenHpIsRecovered` | | healerEntity, recoveryAmount, recoverTagList |
| `whenInitiatingHpRecovery` | | recoverTargetEntity, recoveryAmount, recoverTagList |
| `whenSelfEntersCombat` | B | (fixed outputs only) |
| `whenSelfLeavesCombat` | B | (fixed outputs only) |
| `whenShieldIsAttacked` | | attackerEntity, attackerGuid, unitStatusConfigId, preAttackLayers, postAttackLayers, shieldValueOfThisUnitStatusBeforeAttack, shieldValueOfThisUnitStatusAfterAttack |

---

## 4. Collision

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenEnteringCollisionTrigger` | | enteringEntity, enteringEntityGuid, triggerEntity, triggerEntityGuid, triggerId |
| `whenExitingCollisionTrigger` | | exitingEntity, exitingEntityGuid, triggerEntity, triggerEntityGuid, triggerId |
| `whenOnHitDetectionIsTriggered` | | onHitHurtbox, onHitEntity, onHitLocation |

---

## 5. Variables

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenComplexCreationPresetStatusChanges` | | presetStatusIndex, preChangeValue, postChangeValue |
| `whenCustomVariableChanges` | | variableName, preChangeValue, postChangeValue |
| `whenNodeGraphVariableChanges` | | variableName, preChangeValue, postChangeValue |
| `whenPresetStatusChanges` | | presetStatusId, preChangeValue, postChangeValue |

---

## 6. Status

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenUnitStatusChanges` | | unitStatusConfigId, applierEntity, infiniteDuration, remainingStatusDuration, remainingStatusStacks, originalStatusStacks, slotId |
| `whenUnitStatusEnds` | | unitStatusConfigId, applierEntity, infiniteDuration, remainingStatusDuration, remainingStatusStacks, removerEntity, removalReason, slotId |

---

## 7. Timer

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenGlobalTimerIsTriggered` | | timerName |
| `whenTimerIsTriggered` | | timerName, timerSequenceId, numberOfLoops |

---

## 8. Signal

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `monitorSignal` | | signalSourceEntity + custom args |

> `monitorSignal` receives custom typed arguments. Register with `.onSignal('Name', handler, argDefs)`.

---

## 9. Motion

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenBasicMotionDeviceStops` | | motionDeviceName |
| `whenCreationReachesPatrolWaypoint` | | creationEntity, creationGuid, currentPatrolTemplateId, currentPathIndex, currentReachedWaypointId, nextWaypointId |
| `whenPathReachesWaypoint` | | motionDeviceName, pathPointId |

---

## 10. UI

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenDeckSelectorIsComplete` | | targetPlayer, selectionResultList, completionReason, deckSelectorIndex |
| `whenTabIsSelected` | | tabId, selectorEntity |
| `whenTextBubbleIsCompleted` | | bubbleOwnerEntity, characterEntity, textBubbleConfigurationId, textBubbleCompletionCount |
| `whenUiControlGroupIsTriggered` | | uiControlGroupCompositeIndex, uiControlGroupIndex |

---

## 11. Class & Skill (Beyond)

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenAggroTargetChanges` | B | preChangeAggroTarget, postChangeAggroTarget |
| `whenPlayerClassChanges` | B | preModificationClassConfigId, postModificationConfigId |
| `whenPlayerClassIsRemoved` | B | preModificationClassConfigId, postModificationConfigId |
| `whenPlayerClassLevelChanges` | B | preChangeLevel, postChangeLevel |
| `whenSkillNodeIsCalled` | B | callerEntity, callerGuid, parameter1, parameter2, parameter3 |

---

## 12. Shop & Inventory

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenCustomShopItemIsSold` | | shopOwner, shopOwnerGuid, buyerEntity, shopId, shopItemId, purchaseQuantity |
| `whenItemIsAddedToInventory` | | itemOwnerEntity, itemOwnerGuid, itemConfigId, quantityObtained |
| `whenItemIsLostFromInventory` | | itemOwnerEntity, itemOwnerGuid, itemConfigId, quantityLost |
| `whenItemsInTheInventoryAreUsed` | | itemOwnerEntity, itemOwnerGuid, itemConfigId, amountToUse |
| `whenSellingInventoryItemsInTheShop` | | shopOwner, shopOwnerGuid, buyerEntity, shopId, itemConfigId, purchaseQuantity |
| `whenSellingItemsToTheShop` | | shopOwner, shopOwnerGuid, sellerEntity, shopId, purchaseItemDictionary |
| `whenTheQuantityOfInventoryCurrencyChanges` | | currencyOwnerEntity, currencyOwnerGuid, currencyConfigId, currencyChangeValue |
| `whenTheQuantityOfInventoryItemChanges` | | itemOwnerEntity, itemOwnerGuid, itemConfigId, preChangeQuantity, postChangeQuantity, reasonForChange |

---

## 13. Equipment

| Event | Mode | Key Outputs |
|-------|------|-------------|
| `whenEquipmentAffixValueChanges` | | equipmentOwner, equipmentOwnerGuid, equipmentIndex, affixId, preChangeValue, postChangeValue |
| `whenEquipmentIsEquipped` | | equipmentHolderEntity, equipmentHolderGuid, equipmentIndex |
| `whenEquipmentIsInitialized` | | equipmentOwner, equipmentOwnerGuid, equipmentIndex |
| `whenEquipmentIsUnequipped` | | equipmentOwnerEntity, equipmentOwnerGuid, equipmentIndex |
