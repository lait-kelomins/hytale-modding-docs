# Hytale Entities

> **Index:** [Overview](#overview) | [PlayerRef](#playerref) | [Entity Components](#entity-components) | [Entity Systems](#entity-systems)

---

## Overview

Hytale uses ECS (Entity Component System). Entities are IDs with attached components.

**Key Packages:**
```
com.hypixel.hytale.server.core.entity/              # Entity classes
com.hypixel.hytale.server.core.entity/entities/     # Entity types
com.hypixel.hytale.server.core.entity/entities/player/  # Player-specific
com.hypixel.hytale.server.core.universe/            # PlayerRef, world
```

---

## PlayerRef

The primary way to access player data in the ECS system.

**Class:** `com.hypixel.hytale.server.core.universe.PlayerRef`

### Accessing PlayerRef

```java
// In EntityEventSystem.handle():
PlayerRef playerRef = chunk.getComponent(i, PlayerRef.getComponentType());
String username = playerRef.getUsername();
```

### Key Methods (expected)

```java
playerRef.getUsername()      // Player's username
playerRef.getUUID()          // Player's unique ID
// Additional methods TBD based on actual API
```

---

## Entity Components

Components are data attached to entities. Access via `ArchetypeChunk`.

### Pattern

```java
// Get component type
ComponentType<MyComponent> type = MyComponent.getComponentType();

// Get component from chunk
MyComponent comp = chunk.getComponent(entityIndex, type);
```

### Entity Systems (Location)

```
com.hypixel.hytale.server.core.entity/
├── AnimationUtils.class
├── ChainSyncStorage.class
├── damage/
│   ├── DamageDataComponent       # Damage tracking
│   └── DamageDataSetupSystem
├── effect/
│   ├── ActiveEntityEffect        # Active effects
│   └── EffectControllerComponent # Effect management
└── entities/
    ├── BlockEntity               # Block entities
    └── player/
        ├── CameraManager
        ├── HotbarManager
        ├── HiddenPlayersManager
        ├── MovementManager
        ├── PageManager
        └── data/
            ├── PlayerConfigData
            ├── PlayerDeathPositionData
            ├── PlayerRespawnPointData
            └── PlayerWorldData
```

---

## Player-Specific Classes

**Package:** `com.hypixel.hytale.server.core.entity.entities.player`

| Class | Purpose |
|-------|---------|
| `CameraManager` | Player camera control |
| `HotbarManager` | Hotbar/inventory slots |
| `HiddenPlayersManager` | Player visibility |
| `MovementManager` | Player movement |
| `PageManager` | UI pages |

### Player Data

| Class | Purpose |
|-------|---------|
| `PlayerConfigData` | Player configuration |
| `PlayerDeathPositionData` | Death location tracking |
| `PlayerRespawnPointData` | Respawn location |
| `PlayerWorldData` | World-specific data |
| `UniqueItemUsagesComponent` | Item usage tracking |

---

## Entity Modules

**Package:** `com.hypixel.hytale.server.core.modules.entity`

```
modules/entity/
├── damage/
│   └── RespawnSystems       # Various respawn handlers
├── DespawnComponent         # Entity despawning
├── DespawnSystem
└── player/
    └── PlayerSystems        # Player-specific systems
```

---

## NPC System

**Package:** `com.hypixel.hytale.server.npc`

```
server/npc/
├── asset/builder/         # NPC configuration builders
│   ├── Builder.class
│   ├── BuilderManager.class
│   ├── BuilderFactory.class
│   ├── BuilderComponent.class
│   └── BuilderCombatConfig.class
├── systems/
│   ├── MessageSupportSystem$NPCEntityEventSystem
│   ├── MessageSupportSystem$PlayerEntityEventSystem
│   └── NPCSystems$PrefabPlaceEntityEventSystem
└── animations/
    └── NPCAnimationSlot.class
```

### NPC Interface

**Class:** `com.hypixel.hytale.server.core.universe.world.npc.INonPlayerCharacter`

---

## Entity Spawning

### Prefab System

Entities are spawned via the prefab system:

**Package:** `com.hypixel.hytale.server.core.prefab`

| Class | Purpose |
|-------|---------|
| `PrefabEntry` | Prefab definition |
| `PrefabPlaceEntityEvent` | Event when entity spawned from prefab |
| `PrefabPasteEvent` | Event when prefab pasted |
| `PrefabRotation` | Rotation handling |

> **WARNING - PrefabPlaceEntityEvent:** This event only fires for **structure prefabs**, NOT for NPC/animal spawns. Tested and confirmed to return 0 triggers even when animals are visibly spawning in-game.

> **WARNING - LoadedNPCEvent:** Does **NOT fire** for animal/NPC spawns. Do not rely on this event for detecting new entities.

> **VERIFIED - EntityTickingSystem + NewSpawnComponent:** Use `EntityTickingSystem<EntityStore>` with a query for `NewSpawnComponent` to detect spawns in real-time. The engine adds `NewSpawnComponent` when entities are created and removes it after processing. See [99-api-discoveries.md](api/99-api-discoveries.md) for complete implementation pattern.

> **FALLBACK:** Periodic scanning (every 30 seconds) with `Store.forEachChunk()` can be used if TickingSystem doesn't work for your use case.

### Spawn Interactions

**Class:** `com.hypixel.hytale.server.core.modules.interaction.interaction.config.server.SpawnPrefabInteraction`

Used to spawn entities at a location.

### Spawn Components

| Class | Purpose |
|-------|---------|
| `NewSpawnComponent` | Marks newly spawned entities |
| `FromPrefab` | Marks entities from prefabs |

### Spawn Item Command

**Class:** `com.hypixel.hytale.server.core.modules.item.commands.SpawnItemCommand`

Example of how items are spawned in the world.

---

## LivingEntity

**Class:** `com.hypixel.hytale.server.core.entity.LivingEntity`

Base class for living entities (players, NPCs, animals).

### Related Systems

| Class | Purpose |
|-------|---------|
| `LivingEntityEffectSystem` | Effect handling |
| `LivingEntityEffectClearChangesSystem` | Effect cleanup |

### Related Events

| Event | Purpose |
|-------|---------|
| `LivingEntityInventoryChangeEvent` | Inventory changed |
| `LivingEntityUseBlockEvent` | Block interaction |

---

## Working with Entities

### In Event Handlers

```java
@Override
public void handle(
    int entityIndex,
    ArchetypeChunk<EntityStore> chunk,
    Store<EntityStore> store,
    CommandBuffer<EntityStore> buffer,
    MyEvent event
) {
    // Entity index identifies the entity in the chunk
    // Use chunk.getComponent() to access components

    PlayerRef player = chunk.getComponent(entityIndex, PlayerRef.getComponentType());

    // Use CommandBuffer for deferred operations
    // buffer.createEntity(...);
    // buffer.addComponent(...);
}
```

### Query for Specific Entities

```java
@Override
public Query<EntityStore> getQuery() {
    // Return query to filter entities
    return PlayerRef.getComponentType();  // Only players
}
```

---

## Related Files

- [02-ecs-system.md](api/02-ecs-system.md) - ECS system details
- [05-events.md](api/05-events.md) - Event types
