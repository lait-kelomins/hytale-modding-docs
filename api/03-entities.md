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
├── NPCPlugin                      # Main NPC plugin access
├── entities/
│   └── NPCEntity                  # Base NPC component
├── role/
│   ├── Role                       # Behavior definition
│   └── support/
│       ├── WorldSupport           # World/attitude settings
│       └── StateSupport           # State machine
├── corecomponents/                # Behavior tree nodes
│   ├── Action, ActionBase         # Actions (do something)
│   ├── Sensor, SensorBase         # Sensors (check conditions)
│   └── builders/                  # Config parsers
│       ├── BuilderActionBase
│       └── BuilderSensorBase
├── config/
│   └── AttitudeGroup              # Attitude groupings
├── systems/
│   └── RoleBuilderSystem          # Builds roles on spawn
└── asset/builder/                 # Config builders
    ├── BuilderSupport
    └── holder/                    # Config value holders
        ├── StringArrayHolder
        └── BooleanHolder
```

### NPCPlugin Access

```java
import com.hypixel.hytale.server.npc.NPCPlugin;

// Get NPC plugin instance
NPCPlugin npcPlugin = NPCPlugin.get();

// Spawn NPC by role name
int roleIndex = npcPlugin.getIndex("Cow");
npcPlugin.spawnEntity(store, roleIndex, position, rotation, null, callback);

// Register custom behavior tree components
npcPlugin.registerCoreComponentType("MyAction", BuilderMyAction::new);
npcPlugin.registerCoreComponentType("MySensor", BuilderMySensor::new);
```

### NPCEntity Component

```java
import com.hypixel.hytale.server.npc.entities.NPCEntity;

NPCEntity npc = store.getComponent(entityRef, NPCEntity.getComponentType());
Role role = npc.getRole();
String roleName = npc.getRoleName();  // "Cow", "Pig", etc.

// Remove from spawn population tracking (for tamed animals)
boolean wasTracked = npc.updateSpawnTrackingState(false);
```

### NPC Interface

**Class:** `com.hypixel.hytale.server.core.universe.world.npc.INonPlayerCharacter`

---

## NPC Behavior Trees

NPCs use behavior trees defined in Role JSON files. Custom actions and sensors extend NPC behavior.

### Custom Action

```java
import com.hypixel.hytale.server.npc.corecomponents.ActionBase;
import com.hypixel.hytale.server.npc.role.Role;
import com.hypixel.hytale.server.npc.sensorinfo.InfoProvider;

public class ActionTame extends ActionBase {
    private final Set<String> lovedFood;

    public ActionTame(BuilderActionTame builder, BuilderSupport support) {
        super(builder);
        this.lovedFood = new HashSet<>(Arrays.asList(builder.getLovedFood(support)));
    }

    @Override
    public boolean execute(Ref<EntityStore> ref, Role role, InfoProvider sensorInfo,
                           double dt, Store<EntityStore> store) {
        super.execute(ref, role, sensorInfo, dt, store);

        // Get interacting player
        Ref<EntityStore> playerRef = role.getStateSupport().getInteractionIterationTarget();
        Player player = store.getComponent(playerRef, Player.getComponentType());
        UUIDComponent playerUUID = store.getComponent(playerRef, UUIDComponent.getComponentType());

        // Perform taming logic...
        return true;  // true = success
    }
}
```

### Custom Action Builder

```java
import com.hypixel.hytale.server.npc.corecomponents.builders.BuilderActionBase;
import com.hypixel.hytale.server.npc.asset.builder.holder.StringArrayHolder;
import com.hypixel.hytale.server.npc.asset.builder.BuilderDescriptorState;

public class BuilderActionTame extends BuilderActionBase {
    protected StringArrayHolder lovedFoodHolder = new StringArrayHolder();

    @Override
    public String getShortDescription() { return "Tame the entity"; }

    @Override
    public String getLongDescription() { return getShortDescription(); }

    @Override
    public BuilderDescriptorState getBuilderDescriptorState() {
        return BuilderDescriptorState.Stable;
    }

    @Override
    public ActionTame build(BuilderSupport support) {
        return new ActionTame(this, support);
    }

    @Override
    public Builder<Action> readConfig(JsonElement data) {
        // Parse "Food" array from NPC role JSON
        this.requireStringArray(data, "Food", lovedFoodHolder, 1, Integer.MAX_VALUE,
            null, BuilderDescriptorState.Stable, "Foods for taming", null);
        return super.readConfig(data);
    }

    public String[] getLovedFood(BuilderSupport support) {
        return lovedFoodHolder.get(support.getExecutionContext());
    }
}
```

### Custom Sensor

```java
import com.hypixel.hytale.server.npc.corecomponents.SensorBase;

public class SensorTamed extends SensorBase {
    private final boolean expectedValue;

    public SensorTamed(BuilderSensorTamed builder, BuilderSupport support) {
        super(builder);
        this.expectedValue = builder.getValue(support);
    }

    @Override
    public boolean matches(Ref<EntityStore> ref, Role role, double dt, Store<EntityStore> store) {
        TameComponent tame = store.getComponent(ref, TameComponent.getComponentType());
        if (tame == null) return false;
        return super.matches(ref, role, dt, store) && tame.isTamed() == expectedValue;
    }

    @Override
    public InfoProvider getSensorInfo() { return null; }
}
```

### Custom Sensor Builder

```java
import com.hypixel.hytale.server.npc.corecomponents.builders.BuilderSensorBase;
import com.hypixel.hytale.server.npc.asset.builder.holder.BooleanHolder;

public class BuilderSensorTamed extends BuilderSensorBase {
    protected final BooleanHolder value = new BooleanHolder();

    @Override
    public Sensor build(BuilderSupport support) {
        return new SensorTamed(this, support);
    }

    @Override
    public Builder<Sensor> readConfig(JsonElement data) {
        this.getBoolean(data, "Set", value, true, BuilderDescriptorState.Stable,
            "Whether entity is tamed", null);
        return this;
    }

    public boolean getValue(BuilderSupport support) {
        return value.get(support.getExecutionContext());
    }
}
```

### Registration

```java
@Override
protected void start() {
    NPCPlugin.get().registerCoreComponentType("Tame", BuilderActionTame::new);
    NPCPlugin.get().registerCoreComponentType("Tamed", BuilderSensorTamed::new);
}
```

### NPC Role JSON Usage

Reference in `Server/NPC/Roles/Cow.json`:
```json
{
  "Sensors": {
    "Tamed": { "Set": true }
  },
  "Actions": {
    "Tame": { "Food": ["Wheat", "Carrot"] }
  }
}
```

---

## NPC Attitude System

Controls how NPCs behave toward players.

### Imports
```java
import com.hypixel.hytale.server.core.asset.type.attitude.Attitude;
import com.hypixel.hytale.server.npc.role.support.WorldSupport;
import com.hypixel.hytale.server.npc.config.AttitudeGroup;
```

### Attitude Values
```java
Attitude.REVERED    // Friendly, won't attack, follows
Attitude.NEUTRAL    // Default for most animals
Attitude.HOSTILE    // Will attack player
```

### Getting Attitude
```java
NPCEntity npc = store.getComponent(ref, NPCEntity.getComponentType());
Role role = npc.getRole();
WorldSupport worldSupport = role.getWorldSupport();

// Default attitude (from NPC role config)
Attitude defaultAttitude = worldSupport.getDefaultPlayerAttitude();

// Current attitude toward player
Attitude current = worldSupport.getAttitude(entityRef, playerRef, store);
```

### Setting Attitude (Reflection Required)

The `defaultPlayerAttitude` field is private:

```java
// Cache field once
private static final Field ATTITUDE_FIELD;
static {
    try {
        ATTITUDE_FIELD = WorldSupport.class.getDeclaredField("defaultPlayerAttitude");
        ATTITUDE_FIELD.setAccessible(true);
    } catch (NoSuchFieldException e) {
        throw new RuntimeException(e);
    }
}

// Set to friendly
try {
    ATTITUDE_FIELD.set(worldSupport, Attitude.REVERED);
} catch (IllegalAccessException e) {
    logger.atSevere().log("Failed to set attitude", e);
}
```

### Attitude Groups

Check NPC's attitude group for filtering:

```java
WorldSupport worldSupport = role.getWorldSupport();
AttitudeGroup group = AttitudeGroup.getAssetMap().getAsset(worldSupport.getAttitudeGroup());
String groupId = group.getId();  // "PreyBig", "PreySmall", "Livestock", "Critters"
```

### Common Attitude Groups
| Group | Animals |
|-------|---------|
| `Livestock` | Cow, Pig, Chicken, Sheep |
| `PreyBig` | Deer, Horse, Camel |
| `PreySmall` | Rabbit, Bunny |
| `Critters` | Frog, Mouse, Squirrel |
| `Predator` | Wolf, Bear, Fox |

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
