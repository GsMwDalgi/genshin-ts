# Node Catalog — Extended (Domain-Specific)

Methods called on `f` inside `g.server().on(...)`. Mode column: `B` = beyond only, `C` = classic only, blank = both.

---

## 13. Combat & HP

| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.hpLoss(target, amount, lethal, blockByInvinc, blockByLock, popUpType)` | exec | | deal HP loss (6 params) |
| `f.initiateAttack(target, dmgCoef, dmgIncr, locOff, rotOff, abilityUnit, overwrite, initiator)` | exec | | initiate attack (8 params) |
| `f.recoverHp(target, amount, abilityUnit, overwrite, initiator)` | exec | | recover HP (5 params) |
| `f.recoverHpDirectly(initiator, target, amount, ignoreAdj, aggroMul, aggroInc, healTags)` | exec | | recover HP directly (7 params) |

---

## 14. Effects & Projectiles

| Method | Type | Description |
|--------|------|-------------|
| `f.clearLoopingSpecialEffect(id, entity)` | exec | detach looping VFX |
| `f.clearSpecialEffectsBasedOnSpecialEffectAssets(entity, effectAsset)` | exec | clear VFX (2 params) |
| `f.createProjectile(prefabId, pos, rot, owner, trackTarget, overwriteLvl, lvl, tagList)` | exec | create projectile (8 params) |
| `f.mountLoopingSpecialEffect(effectAsset, target, attachPt, moveWith, rotWith, locOff, rotOff, zoom, sound)` | exec+data | attach looping VFX (9 params, returns instanceId: int) |
| `f.playTimedEffects(effectAsset, target, attachPt, moveWith, rotWith, locOff, rotOff, zoom, sound)` | exec | play timed VFX (9 params) |

---

## 15. Motion & Pathing

| Method | Type | Description |
|--------|------|-------------|
| `f.activateBasicMotionDevice(entity, name)` | exec | start motion device |
| `f.activateDisableFollowMotionDevice(entity, on)` | exec | toggle follow |
| `f.activateFixedPointMotionDevice(entity, name, mode, speed, loc, rot, lockRot, paramType, time)` | exec | move to fixed point (9 params) |
| `f.addTargetOrientedRotationBasedMotionDevice(entity, name, duration, targetAngle)` | exec | add target-oriented rotation (4 params) |
| `f.addUniformBasicLinearMotionDevice(entity, name, duration, velocity)` | exec | add linear motion (4 params) |
| `f.addUniformBasicRotationBasedMotionDevice(entity, name, duration, angVel, axis)` | exec | add rotation motion (5 params) |
| `f.getFollowMotionDeviceTarget(entity)` | data | get follow target (entity + guid) |
| `f.pauseBasicMotionDevice(entity, name)` | exec | pause |
| `f.recoverBasicMotionDevice(entity, name)` | exec | resume |
| `f.stopAndDeleteBasicMotionDevice(entity, name, stopAll)` | exec | stop & delete (3 params) |
| `f.switchFollowMotionDeviceTargetByEntity(entity, followTarget, attachPt, locOff, rotOff, coordSys, followType)` | exec | change follow target (7 params) |
| `f.switchFollowMotionDeviceTargetByGuid(entity, guid, attachPt, locOff, rotOff, coordSys, followType)` | exec | change follow target by guid (7 params) |

---

## 16. Character, Class & Skill (mostly Beyond)

### Skill Management
| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.addCharacterSkill(entity, skillConfigId, skillSlot)` | exec | B | add skill (3 params) |
| `f.deleteCharacterSkillById(entity, configId)` | exec | B | remove by ID |
| `f.deleteCharacterSkillBySlot(entity, skillSlot)` | exec | B | remove by slot (2 params) |
| `f.initializeCharacterSkill(entity, skillSlot)` | exec | B | init skill (2 params) |
| `f.modifyCharacterSkillCd(entity, skillSlot, cdModifier, limitMax)` | exec | B | modify cooldown (4 params) |
| `f.modifySkillCdPercentageBasedOnMaxCd(entity, skillSlot, cdRatioMod, limitMax)` | exec | B | modify CD by % (4 params) |
| `f.modifySkillResourceAmount(entity, resourceConfigId, change)` | exec | B | modify skill resource (3 params) |
| `f.queryCharacterSkill(entity, skillSlot)` | data | B | query skill configId (2 params) |
| `f.setCharacterSkillCd(entity, skillSlot, remainingCd, limitMax)` | exec | B | set cooldown (4 params) |
| `f.setSkillResourceAmount(entity, resourceConfigId, value)` | exec | B | set skill resource (3 params) |

### Class
| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.changePlayerClass(player, configId)` | exec | B | change class |
| `f.changePlayerSCurrentClassLevel(player, level)` | exec | B | set class level |
| `f.increasePlayerSCurrentClassExp(player, exp)` | exec | B | add class EXP |
| `f.queryPlayerClass(player)` | data | B | get class |
| `f.queryPlayerClassLevel(player, configId)` | data | B | get class level |

### Classic-Only
| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.checkClassicModeCharacterId(character)` | data | C | character ID |
| `f.getActiveCharacterOfSpecifiedPlayer(player)` | data | C | active character |
| `f.increasesCharacterSElementalEnergy(entity, amount)` | exec | C | add energy (2 params) |
| `f.setCharacterSElementalEnergy(entity, energy)` | exec | C | set energy |

### Other
| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.modifyingCharacterDisruptorDevice(entity, id)` | exec | B | disruptor device |
| `f.queryCharacterSCurrentMovementSpd(entity)` | data | | movement speed |

---

## 17. Revive & Death

| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.activateRevivePoint(player, id)` | exec | | enable revive point |
| `f.allowForbidPlayerToRevive(player, allow)` | exec | | allow/forbid revive |
| `f.deactivateRevivePoint(player, id)` | exec | | disable revive point |
| `f.defeatAllPlayerSCharacters(player)` | exec | | defeat all |
| `f.getPlayerRemainingRevives(player)` | data | | get revive count |
| `f.getPlayerReviveTime(player)` | data | | get revive timer |
| `f.reviveActiveCharacter(player)` | exec | C | revive active (classic) |
| `f.reviveAllPlayerSCharacters(player, deduct)` | exec | | revive all |
| `f.reviveCharacter(entity)` | exec | B | revive one |
| `f.setPlayerRemainingRevives(player, count)` | exec | | revive limit |
| `f.setPlayerReviveTime(player, duration)` | exec | | revive timer |

---

## 18. Player Queries

| Method | Type | Description |
|--------|------|-------------|
| `f.getAllCharacterEntitiesOfSpecifiedPlayer(player)` | data | player's characters |
| `f.getListOfPlayerEntitiesOnTheField()` | data | all players |
| `f.getPlayerClientInputDeviceType(player)` | data | input device |
| `f.getPlayerEscapeValidity(player)` | data | escape allowed? |
| `f.getPlayerGuidByPlayerId(id)` | data | ID→guid |
| `f.getPlayerIdByPlayerGuid(guid)` | data | guid→ID |
| `f.getPlayerNickname(player)` | data | nickname |
| `f.getPlayerSCurrentUiLayout(player)` | data | current UI layout index |
| `f.queryIfAllPlayerCharactersAreDown(player)` | data | all down? |
| `f.setPlayerEscapeValidity(player, valid)` | exec | allow/forbid escape |

---

## 19. Camera & UI

| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.activateDisableTab(entity, id, on)` | exec | | toggle tab |
| `f.activateUiControlGroupInControlGroupLibrary(player, groupId)` | exec | | activate UI group (2 params) |
| `f.modifyUiControlStatusWithinTheInterfaceLayout(player, controlIdx, status)` | exec | | modify UI control (3 params) |
| `f.removeInterfaceControlGroupFromControlGroupLibrary(player, groupId)` | exec | | remove UI group (2 params) |
| `f.setEntityActiveNameplate(entity, nameplateConfigIdList)` | exec | | nameplate (config_id_list) |
| `f.switchActiveTextBubble(entity, configId)` | exec | | text bubble (config_id) |
| `f.switchCurrentInterfaceLayout(player, index)` | exec | | switch layout |
| `f.switchMainCameraTemplate(players, name)` | exec | B | camera template |

---

## 20. Audio

| Method | Type | Description |
|--------|------|-------------|
| `f.addSoundEffectPlayer(entity, soundIdx, vol, speed, loop, interval, is3d, range, attenuation, attachPt, offset)` | exec | add SFX player (11 params) |
| `f.adjustPlayerBackgroundMusicVolume(player, vol)` | exec | BGM volume |
| `f.adjustSpecifiedSoundEffectPlayer(entity, sfxPlayerId, vol, speed)` | exec | adjust SFX (4 params) |
| `f.closeSpecifiedSoundEffectPlayer(entity, id)` | exec | stop SFX |
| `f.modifyPlayerBackgroundMusic(player, musicIdx, start, end, vol, loop, interval, speed, fadeInOut)` | exec | change BGM (9 params) |
| `f.playerPlaysOneShot2dSoundEffect(player, soundIdx, vol, speed)` | exec | play 2D SFX (4 params) |
| `f.startPausePlayerBackgroundMusic(player, recover)` | exec | play/pause BGM |
| `f.startPauseSpecifiedSoundEffectPlayer(entity, sfxPlayerId, recover)` | exec | pause/resume SFX (3 params) |

---

## 21. Tags

| Method | Type | Description |
|--------|------|-------------|
| `f.addUnitTagToEntity(entity, unitTagIndex)` | exec | add tag (index is int, not str) |
| `f.clearUnitTagsFromEntity(entity)` | exec | clear all tags |
| `f.getEntityListByUnitTag(unitTagIndex)` | data | entities by tag (int index) |
| `f.getEntityUnitTagList(entity)` | data | entity's tags → int_list |
| `f.removeUnitTagFromEntity(entity, unitTagIndex)` | exec | remove tag (index is int) |

---

## 22. Aggro (Beyond)
| Method | Type | Description |
|--------|------|-------------|
| `f.clearSpecifiedTargetSAggroList(owner)` | exec | clear aggro list |
| `f.getAggroListOfCreationInDefaultMode(creation)` | data | creation's default aggro |
| `f.getListOfOwnersThatHaveTheTargetAsTheirAggroTarget(target)` | data | who targets this? |
| `f.getListOfOwnersWhoHaveTheTargetInTheirAggroList(target)` | data | reverse aggro list |
| `f.getTheAggroListOfTheSpecifiedEntity(entity)` | data | aggro list |
| `f.getTheAggroTargetOfTheSpecifiedEntity(owner)` | data | current target |
| `f.queryIfSpecifiedEntityIsInCombat(target)` | data | in combat? |
| `f.queryTheAggroMultiplierOfTheSpecifiedEntity(target)` | data | aggro multiplier (float) |
| `f.queryTheAggroValueOfTheSpecifiedEntity(target, aggroOwner)` | data | aggro value (int) |
| `f.removeTargetEntityFromAggroList(target, owner)` | exec | remove from aggro |
| `f.setTheAggroValueOfSpecifiedEntity(target, aggroOwner, value)` | exec | set aggro value (3 params) |
| `f.tauntTarget(taunter, target)` | exec | force aggro |

---

## 23. Shop & Inventory

### Shop (exec)
| Method | Description |
|--------|-------------|
| `f.addItemsToThePurchaseList(owner, shopId, itemConfigId, currencyDict, purchasable)` | add purchase item (5 params) |
| `f.addNewItemToCustomShopSalesList(owner, shopId, configId, currencyDict, tabId, limitPurchase, limit, priority, canBeSold)` | add custom shop item (9 params, returns shopItemId: int) |
| `f.addNewItemToInventoryShopSalesList(owner, shopId, itemConfigId, currencyDict, tabId, sortPriority, canBeSold)` | add shop item (7 params) |
| `f.closeShop(player)` | close shop |
| `f.modifyInventoryShopItemSalesInfo(owner, shopId, itemConfigId, currencyDict, tabId, priority, canBeSold)` | modify shop item (7 params) |
| `f.modifyItemPurchaseInfoInThePurchaseList(owner, shopId, itemConfigId, currencyDict, purchasable)` | modify purchase info (5 params) |
| `f.openShop(player, owner, id)` | open shop |
| `f.queryCustomShopItemSalesInfo(owner, shopId, shopItemId)` | custom item info (returns struct) |
| `f.queryCustomShopItemSalesList(owner, shopId)` | list custom shop items (returns int_list) |
| `f.queryInventoryShopItemSalesInfo(owner, shopId, itemConfigId)` | shop item info (returns struct) |
| `f.queryInventoryShopItemSalesList(owner, shopId)` | list shop items (returns config_id_list) |
| `f.queryShopItemPurchaseInfo(owner, shopId, itemConfigId)` | purchase info (returns struct) |
| `f.queryShopPurchaseItemList(owner, shopId)` | list purchasable items |
| `f.removeItemFromCustomShopSalesList(owner, shopId, shopItemId)` | remove custom shop item |
| `f.removeItemFromInventoryShopSalesList(owner, shopId, itemConfigId)` | remove shop item (3 params) |
| `f.removeItemFromPurchaseList(owner, shopId, itemConfigId)` | remove purchase item |

### Inventory (exec)
| Method | Description |
|--------|-------------|
| `f.increaseMaximumInventoryCapacity(owner, increase)` | expand capacity (2 params) |
| `f.modifyInventoryCurrencyQuantity(owner, currencyConfigId, change)` | add/remove currency (3 params) |
| `f.modifyInventoryItemQuantity(owner, itemConfigId, change)` | add/remove items (3 params) |

### Inventory Queries (data)
| Method | Returns | Description |
|--------|---------|-------------|
| `f.getAllBasicItemsFromInventory(owner)` | list | all items |
| `f.getAllCurrencyFromInventory(owner)` | dict | all currency (config_id→int) |
| `f.getAllEquipmentFromInventory(owner)` | int_list | all equipment (indexes) |
| `f.getInventoryCapacity(owner)` | int | capacity |
| `f.getInventoryCurrencyQuantity(owner, currencyConfigId)` | int | currency count |
| `f.getInventoryItemQuantity(owner, configId)` | int | item count |

---

## 24. Equipment

| Method | Type | Description |
|--------|------|-------------|
| `f.addAffixToEquipment(equipId, affixConfigId, overwrite, value)` | exec | add affix (4 params, value is float) |
| `f.addAffixToEquipmentAtSpecifiedId(equipId, affixConfigId, insertId, overwrite, value)` | exec | add affix at ID (5 params) |
| `f.getEquipmentAffixConfigId(equipIdx, affixId)` | data | affix config ID |
| `f.getEquipmentAffixList(index)` | data | list affixes |
| `f.getEquipmentAffixValue(index, affixId)` | data | affix value (returns float) |
| `f.getTheEquipmentIndexOfTheSpecifiedEquipmentSlot(entity, row, col)` | data | slot→equip index |
| `f.modifyEquipmentAffixValue(equipIdx, affixId, value)` | exec | change affix value (3 params, value is float) |
| `f.queryEquipmentConfigIdByEquipmentId(index)` | data | config ID |
| `f.queryEquipmentTagList(equipIdx)` | data | equipment tags (config_id_list) |
| `f.removeEquipmentAffix(equipId, affixId)` | exec | remove affix |
| `f.replaceEquipmentToTheSpecifiedSlot(entity, row, col, equipIdx)` | exec | equip item (4 params) |

---

## 25. Loot

| Method | Type | Description |
|--------|------|-------------|
| `f.getAllCurrencyFromLootComponent(dropper)` | data | all loot currency (dict) |
| `f.getAllEquipmentFromLootComponent(loot)` | data | all loot equipment (int_list) |
| `f.getAllItemsFromLootComponent(dropper)` | data | all loot items (dict) |
| `f.getLootComponentCurrencyQuantity(loot, currencyConfigId)` | data | loot currency count |
| `f.getLootComponentItemQuantity(loot, configId)` | data | loot item count |
| `f.modifyLootComponentCurrencyAmount(loot, currencyConfigId, change)` | exec | modify loot currency |
| `f.modifyLootItemComponentQuantity(lootEntity, itemConfigId, change)` | exec | modify loot item (3 params) |
| `f.setInventoryDropItemsCurrencyAmount(owner, currencyConfigId, quantity, lootType)` | exec | set drop currency (4 params) |
| `f.setInventoryItemDropContents(owner, itemDropDict, lootType)` | exec | set drop contents (3 params) |
| `f.setLootDropContent(dropper, lootDropDict)` | exec | set drop contents (dict<config_id, int>) |
| `f.triggerLootDrop(dropper, lootType)` | exec | trigger drop |

---

## 26. Settlement & Ranking

| Method | Type | Description |
|--------|------|-------------|
| `f.getFactionSettlementRankingValue(faction)` | data | get faction ranking |
| `f.getFactionSettlementSuccessStatus(faction)` | data | get faction settlement |
| `f.getPlayerRankingInfo(player)` | data | ranking info (totalScore, winStreak, etc.) |
| `f.getPlayerSettlementRankingValue(player)` | data | get ranking value |
| `f.getPlayerSettlementSuccessStatus(player)` | data | get settlement status |
| `f.setFactionSettlementRankingValue(faction, value)` | exec | faction ranking |
| `f.setFactionSettlementSuccessStatus(faction, status)` | exec | faction settlement (status: `SettlementStatus.Victory` / `.Defeat` / `.Undefined`) |
| `f.setPlayerLeaderboardScoreAsAFloat(playerIdList, score, leaderboardId)` | exec | leaderboard float (3 params, first is int_list) |
| `f.setPlayerLeaderboardScoreAsAnInteger(playerIdList, score, leaderboardId)` | exec | leaderboard int (3 params, first is int_list) |
| `f.setPlayerRankScoreChange(player, status, scoreChange)` | exec | rank change (3 params) |
| `f.setPlayerSettlementRankingValue(player, value)` | exec | ranking score |
| `f.setPlayerSettlementScoreboardDataDisplay(player, order, name, value)` | exec | scoreboard (4 params, value: float/int/str) |
| `f.setPlayerSettlementSuccessStatus(player, status)` | exec | settlement status (2 params, status: `SettlementStatus.Victory` / `.Defeat` / `.Undefined`) |
| `f.switchTheScoringGroupThatAffectsPlayerSCompetitiveRank(player, scoreGroupId)` | exec | switch scoring group |

---

## 27. Environment & Time

| Method | Type | Description |
|--------|------|-------------|
| `f.calculateDayOfTheWeekFromTimestamp(ts)` | data | day of week (int) |
| `f.calculateFormattedTimeFromTimestamp(ts)` | data | timestamp→format |
| `f.calculateTimestampFromFormattedTime(year, month, day, hour, min, sec)` | data | format→timestamp (6 params) |
| `f.modifyEnvironmentSettings(envConfigIdx, playerList, enableWeather, weatherIdx)` | exec | env settings (4 params) |
| `f.queryCurrentEnvironmentTime()` | data | current time |
| `f.queryGameTimeElapsed()` | data | elapsed time |
| `f.queryServerTimeZone()` | data | server timezone (int) |
| `f.queryTimestampUtc0()` | data | UTC timestamp |
| `f.setCurrentEnvironmentTime(time)` | exec | set time |
| `f.setEnvironmentTimePassageSpeed(speed)` | exec | time speed |

---

## 28. Minimap

| Method | Type | Description |
|--------|------|-------------|
| `f.getEntitySMiniMapMarkerStatus(entity)` | data | marker status (full/active/inactive lists) |
| `f.modifyMiniMapMarkerActivationStatus(entity, markerIdList, active)` | exec | toggle marker (3 params) |
| `f.modifyMiniMapZoom(player, zoom)` | exec | zoom level |
| `f.modifyPlayerListForTrackingMiniMapMarkers(entity, markerId, playerList)` | exec | tracking markers (3 params) |
| `f.modifyPlayerListForVisibleMiniMapMarkers(entity, markerId, playerList)` | exec | visible markers (3 params) |
| `f.modifyPlayerMarkersOnTheMiniMap(entity, markerId, player)` | exec | player markers (3 params) |
| `f.querySpecifiedMiniMapMarkerInformation(entity, markerId)` | data | marker info (activation, player lists) |

---

## 29. Patrol & Waypoint

| Method | Type | Description |
|--------|------|-------------|
| `f.getCurrentCreationSPatrolTemplate(entity)` | data | current patrol |
| `f.getPresetPointListByUnitTag(tagId)` | data | preset points by tag |
| `f.getSpecifiedWaypointInfo(pathIdx, waypointId)` | data | waypoint location/orientation |
| `f.getTheNumberOfWaypointsInTheGlobalPath(pathIdx)` | data | waypoint count |
| `f.queryPresetPointPositionRotation(pointIndex)` | data | point pos/rot |
| `f.switchCreationPatrolTemplate(entity, id)` | exec | switch patrol |

---

## 30. Achievement

| Method | Type | Mode | Description |
|--------|------|------|-------------|
| `f.changeAchievementProgressTally(entity, achievementId, amount)` | exec | | modify progress (3 params) |
| `f.queryIfAchievementIsCompleted(entity, id)` | data | | completed? |
| `f.setAchievementProgressTally(entity, achievementId, value)` | exec | | set progress (3 params) |

---

## 31. Scan (B)

| Method | Type | Description |
|--------|------|-------------|
| `f.getTheCurrentlyActiveScanTagConfigId(entity)` | data | get active scan tag |
| `f.setScanComponentSActiveScanTagId(entity, id)` | exec | set active scan tag |
| `f.setScanTagRules(entity, ruleType)` | exec | set scan rules |

---

## 32. Deck Selector

| Method | Type | Description |
|--------|------|-------------|
| `f.closeDeckSelector(player, index)` | exec | close |
| `f.invokeDeckSelector(player, id, duration, resultList, displayList, selectMin, selectMax, refreshMode, refreshMin, refreshMax, defaultReturn)` | exec | show deck selector (11 params) |
| `f.randomDeckSelectorSelectionList(list)` | exec | randomize |

---

## 33. Chat & Channel

| Method | Type | Description |
|--------|------|-------------|
| `f.modifyPlayerChannelPermission(playerGuid, channelIdx, join)` | exec | permissions (3 params) |
| `f.setChatChannelSwitch(index, on)` | exec | toggle channel |
| `f.setPlayerSCurrentChannel(playerGuid, channelIdxList)` | exec | set channel (guid + int_list) |

---

## 34. Gift Box (B)

| Method | Type | Description |
|--------|------|-------------|
| `f.consumeGiftBox(player, giftBoxIdx, quantity)` | exec | consume (3 params) |
| `f.queryCorrespondingGiftBoxConsumption(player, index)` | data | consumed count |
| `f.queryCorrespondingGiftBoxQuantity(player, index)` | data | quantity |

---

## 35. Deployment Group

| Method | Type | Description |
|--------|------|-------------|
| `f.activateDisableEntityDeploymentGroup(groupIdx, activate)` | exec | toggle group (2 params) |
| `f.getCurrentlyActiveEntityDeploymentGroups()` | data | active groups |
