# Reverse Engineering Status - Hytale Server

**Last Updated:** 2026-01-19
**Project:** HytaleServer.jar
**Total Classes:** ~37,900

---

## Current Task

**Focus:** Initial Analysis Complete - All Questions Answered

**Completed:**
1. Player movement and collision systems (no-clip, flying, collision disable) - DONE
2. Spawn systems (player spawn, world spawn configuration) - DONE
3. Interaction system (hotkeys, InteractionType enum) - DONE
4. Inventory architecture (world/instance separation) - DONE

**Next Focus Areas:**
- Deep dive into specific subsystems as needed
- NPC AI behavior tree system
- Combat damage calculation
- Network packet analysis

---

## Completed Analysis

| Package/Class | Status | Notes |
|---------------|--------|-------|
| Entry Points | Identified | `Main.class`, `LateMain.class` |
| Plugin System | Identified | `JavaPlugin`, `EarlyPluginLoader` |
| High-Level Architecture | Mapped | ECS-based, Module system |
| NPC AI System | **COMPLETE** | Blackboard + Action/Sensor architecture |
| Combat & Damage | **COMPLETE** | Damage pipeline with armor/knockback |
| Network Protocol | **COMPLETE** | Packet categories mapped |
| ECS Component System | **COMPLETE** | Store, Ref, Archetype, Components, Systems |
| World Generation | **COMPLETE** | Zone/Biome/Climate/Cave/Prefab systems |
| Movement System | **COMPLETE** | MovementConfig, MovementSettings |
| Collision System | **COMPLETE** | CollisionConfig, CollisionFilter |
| Interaction System | **COMPLETE** | 26 InteractionTypes documented |
| Spawn System | **COMPLETE** | ISpawnProvider implementations |
| Inventory System | **COMPLETE** | Per-world data identified |

---

## Key Architectural Insights

### 1. Entry Point Structure
```
com.hypixel.hytale.Main           -> Bootstrap
com.hypixel.hytale.LateMain       -> Post-plugin-load initialization
com.hypixel.hytale.plugin.early.EarlyPluginLoader -> Plugin loading
com.hypixel.hytale.plugin.early.TransformingClassLoader -> Bytecode transforms
```

### 2. Core Architecture Pattern
- **Entity-Component-System (ECS)**: Uses `EntityStore`, component types, and systems
- **Module-based**: Core functionality split into modules (`modules/collision`, `modules/physics`, `modules/interaction`)
- **Event-driven**: Extensive event system (`event/events/`)
- **Asset-driven configuration**: Game behavior defined via JSON assets

### 3. Package Organization

| Package | Purpose |
|---------|---------|
| `server/core/` | Core server logic |
| `server/core/modules/` | ECS modules (collision, physics, entity, interaction) |
| `server/core/entity/` | Entity definitions (Player, LivingEntity) |
| `server/core/event/` | Event system |
| `server/core/inventory/` | Inventory management |
| `server/core/universe/` | World/Universe management |
| `server/npc/` | NPC system |
| `protocol/` | Network packets |
| `builtin/` | Built-in plugins (instances, blockspawner, creativehub) |
| `math/` | Math utilities (raycast, vectors) |

### 4. Player Movement Architecture
```
MovementManager (player/movement/)
  -> MovementConfig
  -> MovementStatesComponent
  -> PlayerProcessMovementSystem
  -> PlayerMovementManagerSystems
```

### 5. Collision System
```
modules/collision/
  -> CollisionModule
  -> CollisionConfig
  -> CollisionFilter
  -> CharacterCollisionData
  -> HitboxCollision (entity/hitboxcollision/)
```

### 6. Interaction System
```
modules/interaction/
  -> InteractionManager
  -> InteractionChain
  -> InteractionContext
  -> Interaction configs (client/, server/, selector/)
  -> InteractionType (protocol) - Use, Primary, Secondary
```

### 7. Spawn System
```
universe/world/spawn/
  -> ISpawnProvider
  -> GlobalSpawnProvider
  -> IndividualSpawnProvider
  -> FitToHeightMapSpawnProvider

gameplay/respawn/
  -> RespawnController
  -> WorldSpawnPoint
  -> HomeOrSpawnPoint
```

### 8. Instance System (Built-in Plugin)
```
builtin/instances/
  -> InstancesPlugin
  -> InstanceWorldConfig
  -> InstanceEntityConfig
  -> TeleportInstanceInteraction
```

---

## Next Steps

1. **Decompile key classes** to understand:
   - `MovementManager` / `MovementConfig` (flight/no-clip)
   - `CollisionConfig` / `CollisionFilter` (collision disable)
   - `InteractionType` protocol enum (hotkey definitions)
   - `ISpawnProvider` implementations (spawn mechanics)
   - `RaycastSelector` (looking at blocks)
   - `PlayerWorldData` / inventory separation

2. **Answer specific questions** from questions.md

3. **Document patterns** for:
   - Adding custom interactions
   - Modifying player physics
   - Custom spawn logic
   - Inventory management

---

## Questions to Answer (from questions.md)

| # | Question | Status | Answer Location |
|---|----------|--------|-----------------|
| 1 | No-clip mode / creative toggle | **ANSWERED** | ANSWERS.md#q1 |
| 2 | Player flight | **ANSWERED** | ANSWERS.md#q2 |
| 3 | Disable player collisions | **ANSWERED** | ANSWERS.md#q3 |
| 4 | Change default player spawn | **ANSWERED** | ANSWERS.md#q4 |
| 5 | Hotkey system / interactions | **ANSWERED** | ANSWERS.md#q5 |
| 6 | Spawning algorithm | **ANSWERED** | ANSWERS.md#q6 |
| 7 | Get block player is looking at | **ANSWERED** | ANSWERS.md#q7 |
| 8 | Separate inventories between worlds | **ANSWERED** | ANSWERS.md#q8 |
| 9 | Separate inventories between instances | **ANSWERED** | ANSWERS.md#q9 |
| 10 | Player interaction types | **ANSWERED** | ANSWERS.md#q10 |

---

## File References

- **Project Tree:** `docs/PROJECT_TREE.txt`
- **Analysis:** `docs/CODEBASE_ANALYSIS.md`
- **Quick Lookup:** `docs/CODEBASE_ANALYSIS_MAP.md`
- **Answers:** `docs/ANSWERS.md`
- **Decompiled Source:** `source/HytaleServer.jar`
- **Assets:** `source/Assets/`
