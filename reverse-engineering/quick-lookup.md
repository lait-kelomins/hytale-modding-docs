# Codebase Analysis Map - Quick Lookup

**Format:** `[Use Case / Question] -> [File Path] -> [Analysis Tag]`

---

## ECS Core Framework

| Use Case | File Path | Tags |
|----------|-----------|------|
| Entity store management | `component/Store.class` | `#ECS` `#Engine` |
| Entity reference | `component/Ref.class` | `#ECS` `#Entity` |
| Component base interface | `component/Component.class` | `#ECS` |
| Component type token | `component/ComponentType.class` | `#ECS` |
| Entity archetype | `component/Archetype.class` | `#ECS` |
| Component registry | `component/ComponentRegistry.class` | `#ECS` |
| Game entity store | `server/core/universe/world/storage/EntityStore.class` | `#ECS` `#World` |
| System interface | `component/system/ISystem.class` | `#ECS` `#System` |
| Ticking system base | `component/system/tick/TickingSystem.class` | `#ECS` `#System` |
| Entity ticking system | `component/system/tick/EntityTickingSystem.class` | `#ECS` `#System` |
| ECS event base | `component/system/EcsEvent.class` | `#ECS` `#Event` |
| Cancellable ECS event | `component/system/CancellableEcsEvent.class` | `#ECS` `#Event` |
| Query system | `component/query/Query.class` | `#ECS` `#Query` |
| And query | `component/query/AndQuery.class` | `#ECS` `#Query` |
| Spatial indexing | `component/spatial/SpatialSystem.class` | `#ECS` `#Spatial` |
| KD-Tree for queries | `component/spatial/KDTree.class` | `#ECS` `#Spatial` |

---

## Entity Components

| Use Case | File Path | Tags |
|----------|-----------|------|
| Entity position/rotation | `server/core/modules/entity/component/TransformComponent.class` | `#ECS` `#Transform` |
| Entity collision bounds | `server/core/modules/entity/component/BoundingBox.class` | `#ECS` `#Collision` |
| Entity visual model | `server/core/modules/entity/component/ModelComponent.class` | `#ECS` `#Model` |
| Entity velocity | `server/core/modules/physics/component/Velocity.class` | `#ECS` `#Physics` |
| Physics parameters | `server/core/modules/physics/component/PhysicsValues.class` | `#ECS` `#Physics` |
| Movement state | `server/core/entity/movement/MovementStatesComponent.class` | `#ECS` `#Movement` |
| Entity intangible flag | `server/core/modules/entity/component/Intangible.class` | `#ECS` `#Collision` |
| Entity invulnerable flag | `server/core/modules/entity/component/Invulnerable.class` | `#ECS` `#Combat` |
| Entity interactable | `server/core/modules/entity/component/Interactable.class` | `#ECS` `#Interaction` |
| Entity display name | `server/core/modules/entity/component/DisplayNameComponent.class` | `#ECS` `#UI` |
| Entity scale | `server/core/modules/entity/component/EntityScaleComponent.class` | `#ECS` `#Transform` |
| Head rotation | `server/core/modules/entity/component/HeadRotation.class` | `#ECS` `#Transform` |
| Animation state | `server/core/modules/entity/component/ActiveAnimationComponent.class` | `#ECS` `#Animation` |
| Network snapshot | `server/core/modules/entity/component/SnapshotBuffer.class` | `#ECS` `#Network` |
| Damage tracking | `server/core/entity/damage/DamageDataComponent.class` | `#ECS` `#Combat` |
| Dynamic lighting | `server/core/modules/entity/component/DynamicLight.class` | `#ECS` `#Light` |
| Collision results | `server/core/modules/entity/component/CollisionResultComponent.class` | `#ECS` `#Collision` |

---

## Player & Movement

| Use Case | File Path | Tags |
|----------|-----------|------|
| How to make player fly? | `server/core/entity/entities/player/movement/MovementManager.class` | `#Movement` `#Flight` |
| How to enable no-clip? | `server/core/entity/entities/player/movement/MovementConfig.class` | `#Movement` `#Physics` |
| Player movement states | `server/core/entity/movement/MovementStatesComponent.class` | `#Movement` `#State` |
| Process player input | `server/core/modules/entity/player/PlayerInput.class` | `#Movement` `#Input` |
| Movement processing | `server/core/modules/entity/player/PlayerProcessMovementSystem.class` | `#Movement` `#Engine` |
| Player physics | `server/core/modules/physics/component/PhysicsValues.class` | `#Physics` |
| Velocity control | `server/core/modules/physics/component/Velocity.class` | `#Physics` `#Movement` |

---

## Collision

| Use Case | File Path | Tags |
|----------|-----------|------|
| Disable player collision | `server/core/modules/collision/CollisionConfig.class` | `#Collision` |
| Collision filtering | `server/core/modules/collision/CollisionFilter.class` | `#Collision` |
| Block collision | `server/core/modules/collision/BlockCollisionProvider.class` | `#Collision` `#World` |
| Entity collision | `server/core/modules/collision/EntityCollisionProvider.class` | `#Collision` `#Entity` |
| Character collision data | `server/core/modules/collision/CharacterCollisionData.class` | `#Collision` `#Player` |
| Hitbox collision | `server/core/modules/entity/hitboxcollision/HitboxCollision.class` | `#Collision` `#Combat` |
| Make entity intangible | Debug: `EntityIntangibleCommand.class` | `#Collision` `#Command` |

---

## Spawn & Respawn

| Use Case | File Path | Tags |
|----------|-----------|------|
| Change default spawn | `server/core/universe/world/spawn/ISpawnProvider.class` | `#Spawn` `#World` |
| Global spawn point | `server/core/universe/world/spawn/GlobalSpawnProvider.class` | `#Spawn` |
| Per-player spawn | `server/core/universe/world/spawn/IndividualSpawnProvider.class` | `#Spawn` `#Player` |
| World spawn config | `server/core/asset/type/gameplay/SpawnConfig.class` | `#Spawn` `#Config` |
| Respawn controller | `server/core/asset/type/gameplay/respawn/RespawnController.class` | `#Spawn` `#Respawn` |
| World spawn point | `server/core/asset/type/gameplay/respawn/WorldSpawnPoint.class` | `#Spawn` `#World` |
| Player respawn data | `server/core/entity/entities/player/data/PlayerRespawnPointData.class` | `#Spawn` `#Player` |
| Set spawn command | `server/core/universe/world/commands/worldconfig/WorldConfigSetSpawnCommand.class` | `#Spawn` `#Command` |
| NPC spawning | `server/npc/commands/NPCSpawnCommand.class` | `#Spawn` `#NPC` |
| Block spawner plugin | `builtin/blockspawner/BlockSpawnerPlugin.class` | `#Spawn` `#Plugin` |

---

## Interactions & Hotkeys

| Use Case | File Path | Tags |
|----------|-----------|------|
| Interaction types (Use/Primary/Secondary) | `protocol/InteractionType.class` | `#Interaction` `#Hotkeys` |
| Interaction manager | `server/core/entity/InteractionManager.class` | `#Interaction` `#Engine` |
| Interaction chain | `server/core/entity/InteractionChain.class` | `#Interaction` |
| Block interactions | `server/core/modules/interaction/interaction/config/client/UseBlockInteraction.class` | `#Interaction` `#World` |
| Entity interactions | `server/core/modules/interaction/interaction/config/client/UseEntityInteraction.class` | `#Interaction` `#Entity` |
| Interaction configuration | `server/core/modules/interaction/interaction/config/InteractionConfiguration.class` | `#Interaction` `#Config` |
| Mouse button events | `server/core/event/events/player/PlayerMouseButtonEvent.class` | `#Input` `#Event` |
| Interaction type utils | `server/core/modules/interaction/interaction/config/InteractionTypeUtils.class` | `#Interaction` `#Hotkeys` |

---

## Inventory

| Use Case | File Path | Tags |
|----------|-----------|------|
| Main inventory class | `server/core/inventory/Inventory.class` | `#Inventory` |
| Item stacks | `server/core/inventory/ItemStack.class` | `#Inventory` `#Item` |
| Item containers | `server/core/inventory/container/ItemContainer.class` | `#Inventory` `#Container` |
| Inventory commands | `server/core/command/commands/player/inventory/InventoryCommand.class` | `#Inventory` `#Command` |
| Player world data | `server/core/entity/entities/player/data/PlayerWorldData.class` | `#Inventory` `#World` `#State` |
| Inventory packet handler | `server/core/io/handlers/game/InventoryPacketHandler.class` | `#Inventory` `#Net` |
| Modify inventory interaction | `server/core/modules/interaction/interaction/config/server/ModifyInventoryInteraction.class` | `#Inventory` `#Interaction` |

---

## World & Instance

| Use Case | File Path | Tags |
|----------|-----------|------|
| World configuration | `server/core/asset/type/gameplay/WorldConfig.class` | `#World` `#Config` |
| Instance plugin | `builtin/instances/InstancesPlugin.class` | `#Instance` `#Plugin` |
| Instance world config | `builtin/instances/config/InstanceWorldConfig.class` | `#Instance` `#Config` |
| Instance entity config | `builtin/instances/config/InstanceEntityConfig.class` | `#Instance` `#Entity` |
| Teleport to instance | `builtin/instances/interactions/TeleportInstanceInteraction.class` | `#Instance` `#Teleport` |
| Instance exit | `builtin/instances/config/ExitInstance.class` | `#Instance` |
| Add player to world event | `server/core/event/events/player/AddPlayerToWorldEvent.class` | `#World` `#Event` |
| Drain player from world | `server/core/event/events/player/DrainPlayerFromWorldEvent.class` | `#World` `#Event` |

---

## Raycast & Targeting

| Use Case | File Path | Tags |
|----------|-----------|------|
| Get block player looks at | `server/core/modules/interaction/interaction/config/selector/RaycastSelector.class` | `#Raycast` `#Targeting` |
| AABB raycast | `math/raycast/RaycastAABB.class` | `#Raycast` `#Math` |
| Raycast mode | `protocol/RaycastMode.class` | `#Raycast` `#Protocol` |
| Raycast result | `server/core/modules/interaction/interaction/config/selector/RaycastSelector$Result.class` | `#Raycast` |

---

## Game Mode

| Use Case | File Path | Tags |
|----------|-----------|------|
| Game mode enum | `protocol/GameMode.class` | `#GameMode` `#Protocol` |
| Game mode type config | `server/core/asset/type/gamemode/GameModeType.class` | `#GameMode` `#Config` |
| Change game mode event | `server/core/event/events/ecs/ChangeGameModeEvent.class` | `#GameMode` `#Event` |
| Game mode command | `server/core/command/commands/player/GameModeCommand.class` | `#GameMode` `#Command` |
| Creative settings | `server/core/modules/entity/player/PlayerCreativeSettings.class` | `#GameMode` `#Creative` |
| Set game mode packet | `protocol/packets/player/SetGameMode.class` | `#GameMode` `#Net` |

---

## Events

| Use Case | File Path | Tags |
|----------|-----------|------|
| Player connect | `server/core/event/events/player/PlayerConnectEvent.class` | `#Event` `#Player` |
| Player disconnect | `server/core/event/events/player/PlayerDisconnectEvent.class` | `#Event` `#Player` |
| Player ready | `server/core/event/events/player/PlayerReadyEvent.class` | `#Event` `#Player` |
| Mouse button input | `server/core/event/events/player/PlayerMouseButtonEvent.class` | `#Event` `#Input` |
| Mouse motion | `server/core/event/events/player/PlayerMouseMotionEvent.class` | `#Event` `#Input` |
| Break block | `server/core/event/events/ecs/BreakBlockEvent.class` | `#Event` `#World` |
| Place block | `server/core/event/events/ecs/PlaceBlockEvent.class` | `#Event` `#World` |
| Use block | `server/core/event/events/ecs/UseBlockEvent.class` | `#Event` `#World` |
| Inventory change | `server/core/event/events/entity/LivingEntityInventoryChangeEvent.class` | `#Event` `#Inventory` |

---

## Commands (for reference patterns)

| Use Case | File Path | Tags |
|----------|-----------|------|
| Teleport command | `server/core/command/commands/utility/TeleportCommand.class` | `#Command` `#Teleport` |
| Kill command | `server/core/command/commands/player/KillCommand.class` | `#Command` `#Player` |
| Entity remove | `server/core/command/commands/world/entity/EntityRemoveCommand.class` | `#Command` `#Entity` |
| Debug commands | `server/core/command/commands/debug/` | `#Command` `#Debug` |
| Hitbox collision debug | `server/core/command/commands/debug/component/hitboxcollision/` | `#Command` `#Collision` |

---

## Network / Protocol

| Use Case | File Path | Tags |
|----------|-----------|------|
| Set game mode packet | `protocol/packets/player/SetGameMode.class` | `#Net` `#GameMode` |
| Inventory action | `protocol/packets/inventory/InventoryAction.class` | `#Net` `#Inventory` |
| Movement settings | `protocol/MovementSettings.class` | `#Net` `#Movement` |
| Movement states | `protocol/MovementStates.class` | `#Net` `#Movement` |
| Entity spawn | `protocol/packets/entities/` | `#Net` `#Entity` |

---

## Asset Paths

| Asset Type | Path Pattern | Notes |
|------------|--------------|-------|
| RootInteraction | `Server/Item/RootInteractions/{id}.json` | Root interaction definitions |
| Interaction | `Server/Item/Interactions/{id}.json` | Interaction definitions |
| NPC Role | `Server/NPC/Roles/{id}.json` | NPC role definitions |
| Block Type | `Server/Block/Types/{id}.json` | Block definitions |
| Item | `Server/Item/Types/{id}.json` | Item definitions |
| GameMode | `Server/GameMode/{id}.json` | Game mode configs |
| Spawn Config | `Server/Gameplay/SpawnConfig.json` | Spawn configuration |
| World Config | `Server/World/{name}/config.json` | Per-world config |

---

## Quick Question Index

| Question | Primary Class | Secondary Classes |
|----------|---------------|-------------------|
| No-clip mode | `MovementConfig` | `MovementManager`, `PlayerCreativeSettings` |
| Flying | `MovementManager` | `MovementStatesComponent`, `MovementConfig` |
| Disable collision | `CollisionFilter` | `CollisionConfig`, `CharacterCollisionData` |
| Default spawn | `ISpawnProvider` | `GlobalSpawnProvider`, `WorldSpawnPoint` |
| Hotkeys | `InteractionType` | `InteractionTypeUtils`, `PlayerMouseButtonEvent` |
| Spawning algorithm | `BlockSpawnerPlugin` | `ISpawnProvider`, `NPCSpawnCommand` |
| Block player looks at | `RaycastSelector` | `RaycastAABB`, `RaycastMode` |
| World-separate inventory | `PlayerWorldData` | `Inventory`, `AddPlayerToWorldEvent` |
| Instance-separate inventory | `InstanceEntityConfig` | `InstanceWorldConfig`, `PlayerWorldData` |
| Player interactions | `InteractionType` | `UseBlockInteraction`, `UseEntityInteraction` |

---

## NPC AI System

| Use Case | File Path | Tags |
|----------|-----------|------|
| NPC behavior blackboard | `server/npc/blackboard/Blackboard.class` | `#NPC` `#AI` |
| NPC attitude system | `server/npc/blackboard/view/attitude/AttitudeView.class` | `#NPC` `#AI` |
| NPC combat view | `server/npc/blackboard/view/combat/CombatViewSystems.class` | `#NPC` `#Combat` |
| NPC role definition | `server/npc/role/Role.class` | `#NPC` `#AI` |
| NPC action base | `server/npc/corecomponents/ActionBase.class` | `#NPC` `#AI` |
| NPC attack action | `server/npc/corecomponents/combat/ActionAttack.class` | `#NPC` `#Combat` |
| NPC pathfinding | `server/npc/corecomponents/movement/BodyMotionFind.class` | `#NPC` `#Movement` |
| NPC entity detection | `server/npc/corecomponents/entity/SensorEntity.class` | `#NPC` `#AI` |
| NPC player detection | `server/npc/corecomponents/entity/SensorPlayer.class` | `#NPC` `#AI` |
| NPC damage sensor | `server/npc/corecomponents/combat/SensorDamage.class` | `#NPC` `#Combat` |
| Entity filter attitude | `server/npc/corecomponents/entity/filters/EntityFilterAttitude.class` | `#NPC` `#AI` |
| Entity filter line of sight | `server/npc/corecomponents/entity/filters/EntityFilterLineOfSight.class` | `#NPC` `#AI` |
| NPC spawn action | `server/npc/corecomponents/lifecycle/ActionSpawn.class` | `#NPC` `#Spawn` |
| NPC despawn action | `server/npc/corecomponents/lifecycle/ActionDespawn.class` | `#NPC` `#Lifecycle` |
| NPC role builder | `server/npc/role/builders/BuilderRole.class` | `#NPC` `#Builder` |
| NPC steering | `server/npc/movement/Steering.class` | `#NPC` `#Movement` |

---

## Combat & Damage System

| Use Case | File Path | Tags |
|----------|-----------|------|
| Damage event class | `server/core/modules/entity/damage/Damage.class` | `#Combat` `#Damage` |
| Damage cause types | `server/core/modules/entity/damage/DamageCause.class` | `#Combat` `#Damage` |
| Damage calculation | `server/core/modules/entity/damage/DamageCalculatorSystems.class` | `#Combat` `#Damage` |
| Damage processing | `server/core/modules/entity/damage/DamageSystems.class` | `#Combat` `#Damage` |
| Armor damage reduction | `server/core/modules/entity/damage/DamageSystems$ArmorDamageReduction.class` | `#Combat` `#Armor` |
| Fall damage players | `server/core/modules/entity/damage/DamageSystems$FallDamagePlayers.class` | `#Combat` `#Damage` |
| Fall damage NPCs | `server/core/modules/entity/damage/DamageSystems$FallDamageNPCs.class` | `#Combat` `#Damage` |
| Knockback system | `server/core/entity/knockback/KnockbackComponent.class` | `#Combat` `#Physics` |
| Combat data tracking | `server/core/entity/damage/DamageDataComponent.class` | `#Combat` `#State` |

---

## NPC Quick Question Index

| Question | Primary Class | Secondary Classes |
|----------|---------------|-------------------|
| How does NPC AI work? | `Blackboard` | `Role`, `ActionBase`, `SensorEntity` |
| How to make NPC attack? | `ActionAttack` | `SensorTarget`, `CombatSupport` |
| How to customize NPC behavior? | `Role` | `BuilderRole`, NPC asset JSONs |
| How does NPC pathfinding work? | `BodyMotionFind` | `Steering`, `SteeringForceAvoidCollision` |
| How to make NPC detect players? | `SensorPlayer` | `EntityFilterLineOfSight`, `AttitudeView` |
| How does damage work? | `Damage` | `DamageCause`, `DamageSystems` |
| How to modify damage calculation? | `DamageCalculatorSystems` | `DamageSystems$ArmorDamageReduction` |
| How does knockback work? | `KnockbackComponent` | `KnockbackSystems` |

---

## World Generation System

| Use Case | File Path | Tags |
|----------|-----------|------|
| World generation interface | `server/core/universe/world/worldgen/IWorldGen.class` | `#WorldGen` `#Engine` |
| World gen provider interface | `server/core/universe/world/worldgen/provider/IWorldGenProvider.class` | `#WorldGen` `#Provider` |
| Flat world generator | `server/core/universe/world/worldgen/provider/FlatWorldGenProvider.class` | `#WorldGen` `#Flat` |
| Void world generator | `server/core/universe/world/worldgen/provider/VoidWorldGenProvider.class` | `#WorldGen` `#Void` |
| Main chunk generator | `server/worldgen/chunk/ChunkGenerator.class` | `#WorldGen` `#Chunk` |
| Generated chunk data | `server/core/universe/world/worldgen/GeneratedChunk.class` | `#WorldGen` `#Chunk` |
| Zone definition | `server/worldgen/zone/Zone.class` | `#WorldGen` `#Zone` |
| Zone pattern generator | `server/worldgen/zone/ZonePatternGenerator.class` | `#WorldGen` `#Zone` |
| Biome base class | `server/worldgen/biome/Biome.class` | `#WorldGen` `#Biome` |
| Custom biome | `server/worldgen/biome/CustomBiome.class` | `#WorldGen` `#Biome` |
| Biome pattern generator | `server/worldgen/biome/BiomePatternGenerator.class` | `#WorldGen` `#Biome` |
| Climate noise | `server/worldgen/climate/ClimateNoise.class` | `#WorldGen` `#Climate` |
| Climate graph | `server/worldgen/climate/ClimateGraph.class` | `#WorldGen` `#Climate` |
| Cave generator | `server/worldgen/cave/CaveGenerator.class` | `#WorldGen` `#Cave` |
| Cave type | `server/worldgen/cave/CaveType.class` | `#WorldGen` `#Cave` |
| Prefab loading cache | `server/worldgen/prefab/PrefabLoadingCache.class` | `#WorldGen` `#Prefab` |
| Prefab placement | `server/worldgen/prefab/PrefabPasteUtil.class` | `#WorldGen` `#Prefab` |
| Block populator | `server/worldgen/chunk/populator/BlockPopulator.class` | `#WorldGen` `#Populator` |
| Cave populator | `server/worldgen/chunk/populator/CavePopulator.class` | `#WorldGen` `#Cave` |
| Worldgen plugin | `builtin/worldgen/WorldGenPlugin.class` | `#WorldGen` `#Plugin` |
| Worldgen commands | `server/core/command/commands/world/worldgen/WorldGenCommand.class` | `#WorldGen` `#Command` |

---

## World Generation Quick Question Index

| Question | Primary Class | Secondary Classes |
|----------|---------------|-------------------|
| How does world generation work? | `ChunkGenerator` | `IWorldGen`, `GeneratedChunk` |
| How to create custom world gen? | `IWorldGenProvider` | `FlatWorldGenProvider` |
| How do biomes work? | `Biome` | `BiomePatternGenerator`, `BiomeInterpolation` |
| How do zones work? | `Zone` | `ZonePatternGenerator`, `ZonePatternProvider` |
| How do caves generate? | `CaveGenerator` | `CaveType`, `CaveNode` |
| How does climate affect biomes? | `ClimateNoise` | `ClimateGraph`, `ClimateType` |
| How are structures placed? | `PrefabPopulator` | `PrefabContainer`, `PrefabLoadingCache` |
| How does biome blending work? | `BiomeInterpolation` | `FadeContainer` |
| How to customize terrain layers? | `LayerContainer` | `Biome.layerContainer` |
| How to add surface decoration? | `CoverContainer` | `BlockPopulator` |

---

## ECS Quick Question Index

| Question | Primary Class | Secondary Classes |
|----------|---------------|-------------------|
| How does ECS work? | `Store` | `Ref`, `Component`, `Archetype` |
| How to get entity component? | `Store.getComponent()` | `ComponentType.getComponentType()` |
| How to add component to entity? | `Store.addComponent()` | `Store.ensureAndGetComponent()` |
| How to iterate entities? | `Store.forEachChunk()` | `Query`, `ArchetypeChunk` |
| How to listen for entity events? | `EntityEventSystem` | `Store.invoke()` |
| How to create custom system? | `TickingSystem` | `ISystem`, `SystemDependency` |
| How to query entities by components? | `Query` | `AndQuery`, `OrQuery`, `NotQuery` |
| How to find nearby entities? | `SpatialSystem` | `KDTree`, `SpatialResource` |
| How to get entity position? | `TransformComponent` | `getPosition()`, `setPosition()` |
| How to modify entity velocity? | `Velocity` | `addForce()`, `set()`, `addInstruction()` |
| How to make entity intangible? | `Intangible` | `Store.addComponent()` |
| How to make entity invulnerable? | `Invulnerable` | `Store.addComponent()` |
