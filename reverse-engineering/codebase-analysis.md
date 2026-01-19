# Hytale Server Codebase Analysis

**Version:** Initial Analysis
**Date:** 2026-01-19

---

## Table of Contents

1. [Entry Points](#entry-points)
2. [Plugin System](#plugin-system)
3. [Entity-Component-System](#entity-component-system)
4. [Player Systems](#player-systems)
5. [Movement & Physics](#movement--physics)
6. [Collision System](#collision-system)
7. [Interaction System](#interaction-system)
8. [Spawn System](#spawn-system)
9. [Inventory System](#inventory-system)
10. [World & Instance System](#world--instance-system)
11. [Raycast System](#raycast-system)
12. [Event System](#event-system)

---

## Entry Points
`#Engine` `#Bootstrap`

### Main Entry Point

**Path:** `com/hypixel/hytale/Main.class`

The server bootstrap sequence:
1. `Main.main()` - JVM entry point
2. `EarlyPluginLoader` - Loads plugins before server init
3. `TransformingClassLoader` - Applies bytecode transforms
4. `LateMain` - Post-plugin initialization
5. `HytaleServer` - Core server instance

**Side Effects:**
- Sets up class loading hierarchy
- Initializes plugin system
- Registers early transformers

**Pattern Example:**
```java
// Conceptual bootstrap flow
public class Main {
    public static void main(String[] args) {
        EarlyPluginLoader.loadEarlyPlugins();
        TransformingClassLoader classLoader = new TransformingClassLoader();
        // Launch server with transformed classes
        LateMain.start(classLoader);
    }
}
```

---

## Plugin System
`#Hooks` `#Plugins`

### JavaPlugin Base Class

**Path:** `com/hypixel/hytale/server/core/plugin/`

Plugins extend `JavaPlugin` with lifecycle methods:
- `setup()` - Early initialization
- `start()` - Command/event registration
- `shutdown()` - Cleanup

**Side Effects:**
- Plugins can register events
- Plugins can add commands
- Plugins can transform classes (early plugins)

**Pattern Example:**
```java
public class MyPlugin extends JavaPlugin {
    public MyPlugin(JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        // Early init - before server fully started
    }

    @Override
    protected void start() {
        // Register commands, events
        getEventRegistry().register(PlayerConnectEvent.class, this::onConnect);
        getCommandRegistry().registerCommand(new MyCommand());
    }

    @Override
    protected void shutdown() {
        // Cleanup
    }
}
```

---

## Entity-Component-System
`#Engine` `#ECS`

### Core ECS Architecture

**Key Packages:**
- `com.hypixel.hytale.component` - Base ECS framework
- `com.hypixel.hytale.server.core.modules.entity.component` - Entity components
- `com.hypixel.hytale.server.core.modules.physics.component` - Physics components
- `com.hypixel.hytale.server.core.modules.collision` - Collision system
- `com.hypixel.hytale.server.core.universe.world.storage` - World/Entity storage

### ECS Core Classes

| Class | Path | Purpose |
|-------|------|---------|
| `Store<ECS_TYPE>` | `component.Store` | Main entity store managing refs and components |
| `Ref<ECS_TYPE>` | `component.Ref` | Reference to entity data (can become stale) |
| `Component<ECS_TYPE>` | `component.Component` | Interface for all components |
| `ComponentType<ECS_TYPE, T>` | `component.ComponentType` | Type token for component registration |
| `Archetype<ECS_TYPE>` | `component.Archetype` | Collection of component types defining entity shape |
| `ComponentRegistry<ECS_TYPE>` | `component.ComponentRegistry` | Registry for all component types |
| `EntityStore` | `universe.world.storage.EntityStore` | Concrete store for game entities |

### Archetype System

Archetypes define the "shape" of an entity (which components it has):

```java
// Archetype represents a unique combination of components
public class Archetype<ECS_TYPE> implements Query<ECS_TYPE> {
    private final ComponentType<ECS_TYPE, ?>[] componentTypes;

    // Create archetype with components
    Archetype<ECS_TYPE> of(ComponentType... types);

    // Add/remove components dynamically changes archetype
    Archetype<ECS_TYPE> add(Archetype<ECS_TYPE>, ComponentType<ECS_TYPE, T>);
    Archetype<ECS_TYPE> remove(Archetype<ECS_TYPE>, ComponentType<ECS_TYPE, T>);

    boolean contains(ComponentType<ECS_TYPE, ?>);
}
```

### Store Operations

```java
// Entity Store API
public class Store<ECS_TYPE> implements ComponentAccessor<ECS_TYPE> {
    // Entity management
    Ref<ECS_TYPE> addEntity(Archetype<ECS_TYPE>, AddReason);
    Holder<ECS_TYPE> removeEntity(Ref<ECS_TYPE>, RemoveReason);
    Holder<ECS_TYPE> copyEntity(Ref<ECS_TYPE>);

    // Component access
    <T> T getComponent(Ref<ECS_TYPE>, ComponentType<ECS_TYPE, T>);
    <T> T addComponent(Ref<ECS_TYPE>, ComponentType<ECS_TYPE, T>);
    <T> T ensureAndGetComponent(Ref<ECS_TYPE>, ComponentType<ECS_TYPE, T>);
    <T> void removeComponent(Ref<ECS_TYPE>, ComponentType<ECS_TYPE, T>);

    // Iteration
    void forEachChunk(Query<ECS_TYPE>, BiConsumer<ArchetypeChunk, CommandBuffer>);
    void forEachEntityParallel(Query<ECS_TYPE>, IntBiObjectConsumer<...>);

    // Events
    <Event> void invoke(Ref<ECS_TYPE>, Event);  // Entity event
    <Event> void invoke(Event);                  // World event

    // Ticking
    void tick(float deltaTime);
}
```

### Entity Components

**Core Components (`modules/entity/component/`):**

| Component | Purpose | Key Fields |
|-----------|---------|------------|
| `TransformComponent` | Position & rotation | `position: Vector3d`, `rotation: Vector3f`, `chunk: WorldChunk` |
| `BoundingBox` | Collision bounds | `boundingBox: Box`, `detailBoxes: Map<String, DetailBox[]>` |
| `ModelComponent` | Visual model reference | `model: Model`, `isNetworkOutdated: boolean` |
| `ActiveAnimationComponent` | Current animation | Animation state |
| `AudioComponent` | Sound emitter | Audio configuration |
| `DisplayNameComponent` | Entity name display | Name string |
| `EntityScaleComponent` | Entity size | Scale factor |
| `HeadRotation` | Head look direction | Head rotation angles |
| `Intangible` | No collision marker | Flag component |
| `Interactable` | Can be interacted with | Interaction settings |
| `Invulnerable` | Cannot take damage | Flag component |
| `SnapshotBuffer` | Network state buffer | State snapshots |
| `FromPrefab` | Spawned from prefab | Prefab reference |
| `FromWorldGen` | Generated by worldgen | Generation marker |
| `WorldGenId` | World gen identifier | ID reference |
| `NewSpawnComponent` | Recently spawned | Spawn flag |
| `PersistentModel` | Model persists | Persistence flag |
| `PersistentDynamicLight` | Light persists | Light settings |
| `DynamicLight` | Dynamic lighting | Light parameters |
| `PropComponent` | Decorative prop | Prop settings |
| `RotateObjectComponent` | Rotating entity | Rotation parameters |
| `RespondToHit` | Hit response behavior | Response config |
| `HiddenFromAdventurePlayers` | Hidden in adventure | Visibility flag |
| `CollisionResultComponent` | Collision results | Latest collision data |
| `PositionDataComponent` | Position tracking | Position history |
| `MovementAudioComponent` | Movement sounds | Audio triggers |

**Physics Components (`modules/physics/component/`):**

| Component | Purpose | Key Methods |
|-----------|---------|-------------|
| `Velocity` | Entity velocity | `getVelocity()`, `setVelocity()`, `addForce()`, `addInstruction()` |
| `PhysicsValues` | Physics parameters | Gravity, friction, etc. |

**Movement Components (`entity/movement/`):**

| Component | Purpose | Notes |
|-----------|---------|-------|
| `MovementStatesComponent` | Current movement state | Flying, walking, sprinting, etc. |

**Damage Components (`entity/damage/`):**

| Component | Purpose | Notes |
|-----------|---------|-------|
| `DamageDataComponent` | Damage tracking | Combat history |

### ECS Systems

**System Interface:**
```java
public interface ISystem<ECS_TYPE> {
    void onSystemRegistered();
    void onSystemUnregistered();
    SystemGroup<ECS_TYPE> getGroup();
    Set<Dependency<ECS_TYPE>> getDependencies();
}
```

**System Types:**
- `TickingSystem` - Runs every tick
- `ArchetypeTickingSystem` - Ticks entities by archetype
- `EntityTickingSystem` - Per-entity tick
- `DelayedSystem` - Delayed execution
- `EntityEventSystem` - Entity event handlers
- `EventSystem` - World event handlers
- `QuerySystem` - Query-based systems
- `RefSystem` - Reference tracking systems
- `RefChangeSystem` - Reference change detection

**Query System:**
```java
// Query types for filtering entities
AndQuery    // All conditions must match
OrQuery     // Any condition matches
NotQuery    // Condition must not match
AnyQuery    // Matches any archetype
ExactArchetypeQuery // Exact archetype match
ReadWriteArchetypeQuery // Read/write access query
```

### Component Dependency System

**Path:** `component/dependency/`

Systems declare dependencies for proper execution order:
- `Dependency` - Base dependency
- `SystemDependency` - System-to-system dependency
- `SystemGroupDependency` - Group-level dependency
- `OrderPriority` - Execution priority
- `DependencyGraph` - Dependency resolution

### Spatial Indexing

**Path:** `component/spatial/`

Efficient spatial queries for nearby entities:
- `SpatialSystem` - Spatial indexing system
- `SpatialResource` - Spatial data storage
- `KDTree` - K-dimensional tree for queries
- `MortonCode` - Space-filling curve for indexing

**Side Effects:**
- Component changes trigger system updates
- Entity references can become stale on despawn
- Systems run on tick cycles
- Archetype changes cause entity migration between chunks

**Pattern Example:**
```java
// Getting a component from an entity
Object componentType = ComponentClass.getComponentType();
Object component = store.getComponent(entityRef, componentType);

// Ensuring a component exists
Object component = store.ensureAndGetComponent(entityRef, componentType);

// Stale ref handling
try {
    component = store.getComponent(entityRef, componentType);
} catch (Exception e) {
    if (e.getMessage().contains("Invalid entity")) {
        // Entity despawned - clean up tracking
    }
}

// Velocity manipulation
Velocity velocity = store.getComponent(entityRef, Velocity.getComponentType());
velocity.addForce(0, 10, 0);           // Add upward force
velocity.set(0, 5, 0);                  // Set velocity directly
velocity.addInstruction(force, config, type); // Queued instruction

// Transform manipulation
TransformComponent transform = store.getComponent(ref, TransformComponent.getComponentType());
Vector3d pos = transform.getPosition();
transform.setPosition(new Vector3d(x, y, z));
transform.teleportPosition(new Vector3d(x, y, z)); // Network-synced teleport
```

---

## Player Systems
`#Player` `#Entity`

### Player Entity

**Path:** `com/hypixel/hytale/server/core/entity/entities/Player.class`

Player inherits from `LivingEntity` and has specialized managers:
- `MovementManager` - Controls player movement
- `HotbarManager` - Hotbar slot management
- `CameraManager` - Camera control
- `PageManager` - UI page system
- `WindowManager` - Inventory windows
- `HudManager` - HUD elements

**Key Data Classes:**
- `PlayerConfigData` - Player configuration
- `PlayerWorldData` - Per-world player data
- `PlayerRespawnPointData` - Respawn location
- `PlayerDeathPositionData` - Death location

**Pattern Example:**
```java
Player player = (Player) event.getPlayer();

// Access inventory
Inventory inventory = ((LivingEntity) player).getInventory();

// Access movement
MovementManager movement = player.getMovementManager();

// Send updates
player.sendInventory();
```

---

## Movement & Physics
`#Movement` `#Physics` `#Flight`

### Movement System

**Path:** `com/hypixel/hytale/server/core/entity/entities/player/movement/`

**Key Classes:**
- `MovementManager` - Core movement controller
- `MovementConfig` - Movement parameters
- `MovementStatesComponent` - Current movement state

**Path:** `com/hypixel/hytale/server/core/modules/physics/`

**Physics Classes:**
- `PhysicsValues` - Physics parameters
- `Velocity` - Velocity component with instructions
- `SimplePhysicsProvider` - Basic physics calculations
- `ForceAccumulator` - Force calculations

**Movement States (from protocol):**
- `MovementStates` - Enum of movement states
- `MovementType` - Movement type identifiers

**Side Effects:**
- Movement updates propagate to clients
- Physics affects collision detection
- Velocity instructions queue force changes

**Pattern Example:**
```java
// Conceptual movement modification
MovementConfig config = player.getMovementConfig();
// Config likely contains: speed, jump height, gravity, flight ability

// Physics modification
Velocity velocity = store.getComponent(entityRef, Velocity.getComponentType());
velocity.addInstruction(Velocity.Instruction.SET_Y, 10.0); // Upward velocity

// Movement states
MovementStatesComponent states = getMovementStates(player);
// States likely include: FLYING, WALKING, SPRINTING, SNEAKING, etc.
```

---

## Collision System
`#Collision` `#Physics`

### Collision Module

**Path:** `com/hypixel/hytale/server/core/modules/collision/`

**Key Classes:**
- `CollisionModule` - Main collision system
- `CollisionConfig` - Collision configuration
- `CollisionFilter` - Filters what collides with what
- `CharacterCollisionData` - Character-specific collision
- `HitboxCollision` - Hitbox-based collision

**Collision Types:**
- Block collision (`BlockCollisionData`, `BlockCollisionProvider`)
- Entity collision (`EntityCollisionProvider`, `EntityContactData`)
- Box collision (`BoxCollisionData`)

**Side Effects:**
- Collision affects movement
- Collision triggers contact events
- Collision can be filtered per-entity

**Pattern Example:**
```java
// Conceptual collision disable
// Option 1: Use CollisionFilter
CollisionFilter filter = getCollisionFilter(player);
filter.setCollisionEnabled(false);

// Option 2: Make entity intangible
// See EntityIntangibleCommand for pattern

// Option 3: Modify collision config
CollisionConfig config = getCollisionConfig(player);
// Config likely has flags for block/entity collision
```

### Hitbox Collision Commands

**Path:** `com/hypixel/hytale/server/core/command/commands/debug/component/hitboxcollision/`

These debug commands show the hitbox collision system can be added/removed:
- `HitboxCollisionAddCommand`
- `HitboxCollisionRemoveCommand`

---

## Interaction System
`#Interaction` `#Input` `#Hotkeys`

### Interaction Architecture

**Path:** `com/hypixel/hytale/server/core/modules/interaction/`

**Key Classes:**
- `InteractionManager` - Manages interaction chains
- `InteractionChain` - Sequence of interactions
- `InteractionContext` - Context for current interaction
- `Interaction` - Base interaction class

### InteractionType Enum
`#Protocol` `#Hotkeys`

**Path:** `com/hypixel/hytale/protocol/InteractionType.class`

This enum defines the available interaction types that map to hotkeys:
- `Use` - Right-click / E key (likely)
- `Primary` - Left-click / attack
- `Secondary` - Alternative action

**Interaction Configurations:**

Located in `interaction/config/`:
- `client/` - Client-side interactions (block place, break, use)
- `server/` - Server-side interactions (spawn, modify, teleport)
- `selector/` - Target selection (RaycastSelector)
- `none/` - Non-targeted interactions

**Key Interaction Types:**
- `UseBlockInteraction` - Using blocks
- `UseEntityInteraction` - Using entities
- `PlaceBlockInteraction` - Placing blocks
- `BreakBlockInteraction` - Breaking blocks
- `WieldingInteraction` - Weapon/tool wielding

**Side Effects:**
- Interactions can chain together
- Interactions modify world state
- Interactions trigger events

**Pattern Example:**
```java
// Setting up an entity interaction
Object interactions = store.ensureAndGetComponent(entityRef, interactionsCompType);

Class<?> interactionTypeClass = Class.forName("com.hypixel.hytale.protocol.InteractionType");
Object useType = Arrays.stream(interactionTypeClass.getEnumConstants())
    .filter(e -> e.toString().equals("Use")).findFirst().orElse(null);

Method setIntId = interactions.getClass().getMethod("setInteractionId",
    interactionTypeClass, String.class);
setIntId.invoke(interactions, useType, "Root_MyInteraction");
```

### PlayerMouseButtonEvent
`#Event` `#Input`

**Path:** `com/hypixel/hytale/server/core/event/events/player/PlayerMouseButtonEvent.class`

Captures player mouse input:
- Button type (MouseButtonType enum)
- Target entity (nullable)
- Item in hand
- Block target (likely)

---

## Spawn System
`#Spawn` `#World`

### Spawn Providers

**Path:** `com/hypixel/hytale/server/core/universe/world/spawn/`

**Key Classes:**
- `ISpawnProvider` - Interface for spawn logic
- `GlobalSpawnProvider` - Single spawn for all players
- `IndividualSpawnProvider` - Per-player spawns
- `FitToHeightMapSpawnProvider` - Spawn at terrain height

### Respawn System

**Path:** `com/hypixel/hytale/server/core/asset/type/gameplay/respawn/`

**Key Classes:**
- `RespawnController` - Controls respawn behavior
- `WorldSpawnPoint` - World's spawn location
- `HomeOrSpawnPoint` - Player home or world spawn

### Spawn Configuration

**Path:** `com/hypixel/hytale/server/core/asset/type/gameplay/SpawnConfig.class`

Configures spawning behavior via assets.

### World Spawn Commands

**Path:** `com/hypixel/hytale/server/core/universe/world/commands/worldconfig/`

- `WorldConfigSetSpawnCommand`
- `WorldConfigSetSpawnDefaultCommand`

**Side Effects:**
- Spawn changes affect new player joins
- Respawn uses configured spawn points
- Individual spawns stored per-player

**Pattern Example:**
```java
// Conceptual spawn modification
// Option 1: Through world config
WorldConfig config = world.getConfig();
config.setSpawnPoint(x, y, z);

// Option 2: Through RespawnController
RespawnController controller = getRespawnController();
controller.setWorldSpawn(position);

// Option 3: Per-player spawn
PlayerRespawnPointData respawnData = player.getRespawnPointData();
respawnData.setRespawnPoint(position);
```

---

## Inventory System
`#Inventory` `#State`

### Core Inventory

**Path:** `com/hypixel/hytale/server/core/inventory/`

**Key Classes:**
- `Inventory` - Main inventory class
- `ItemStack` - Stack of items
- `ItemContainer` - Container interface
- `ItemContext` - Item operation context

### Container Types
- `SimpleItemContainer`
- `CombinedItemContainer`
- `DelegateItemContainer`
- `ItemStackItemContainer`

### Inventory Transactions

**Path:** `com/hypixel/hytale/server/core/inventory/transaction/`

Supports transactional inventory operations:
- `MoveTransaction`
- `ClearTransaction`
- `ItemStackTransaction`

### Player World Data
`#State` `#Persistence`

**Path:** `com/hypixel/hytale/server/core/entity/entities/player/data/PlayerWorldData.class`

Stores per-world player data - **key for inventory separation**.

**Side Effects:**
- Inventory changes require `markChanged()` and `sendInventory()`
- Transactions can be rolled back
- Per-world data enables world-specific inventories

**Pattern Example:**
```java
// Inventory manipulation
Inventory inventory = ((LivingEntity) player).getInventory();
byte activeSlot = inventory.getActiveHotbarSlot();

// Remove item
inventory.getHotbar().removeItemStackFromSlot((short) activeSlot, 1);
inventory.markChanged();
player.sendInventory();

// Per-world data (conceptual)
PlayerWorldData worldData = player.getWorldData(worldId);
// worldData likely contains inventory state per-world
```

---

## World & Instance System
`#World` `#Instance`

### World Management

**Path:** `com/hypixel/hytale/server/core/universe/world/`

**Key Classes:**
- `WorldConfig` - World configuration
- `SpawnUtil` - Spawn utilities

### Instance Plugin (Built-in)

**Path:** `com/hypixel/hytale/builtin/instances/`

Provides instance (dungeon-style) world support:
- `InstancesPlugin` - Main plugin
- `InstanceWorldConfig` - Instance configuration
- `InstanceEntityConfig` - Entity config for instances
- `TeleportInstanceInteraction` - Teleport to instance

**Instance Features:**
- Separate worlds per instance
- Instance blocks for triggers
- Exit/return mechanics
- Timeout-based cleanup

**Side Effects:**
- Instances are isolated worlds
- Player data may be separate per instance
- Instances can auto-cleanup

**Pattern Example:**
```java
// Instance interaction (conceptual)
TeleportInstanceInteraction interaction = new TeleportInstanceInteraction();
interaction.setInstanceWorld("DungeonTemplate");
interaction.setOriginSource(OriginSource.PLAYER);
```

---

## Raycast System
`#Raycast` `#Targeting`

### Raycast Classes

**Path:** `com/hypixel/hytale/math/raycast/`

**Key Classes:**
- `RaycastAABB` - Axis-aligned bounding box raycast
- `RaycastConsumer` - Callback for raycast hits

### Interaction Raycasting

**Path:** `com/hypixel/hytale/server/core/modules/interaction/interaction/config/selector/`

**Key Classes:**
- `RaycastSelector` - Selects targets via raycast
- `RaycastSelector.Result` - Raycast result

**Protocol:**
- `RaycastMode` - Raycast mode enum
- `RaycastSelector` - Protocol definition

**Side Effects:**
- Raycasts find blocks/entities in view
- Results include hit position and target
- Used by interaction system for targeting

**Pattern Example:**
```java
// Conceptual raycast to find looked-at block
// Option 1: Through interaction system
RaycastSelector selector = new RaycastSelector();
selector.setMode(RaycastMode.BLOCK);
RaycastSelector.Result result = selector.select(player, range);
BlockPosition lookingAt = result.getBlockPosition();

// Option 2: Direct raycast
Vector3d eyePos = player.getEyePosition();
Vector3d direction = player.getLookDirection();
RaycastAABB.raycast(eyePos, direction, maxDistance, consumer);
```

---

## Event System
`#Event` `#Hooks`

### Event Architecture

**Path:** `com/hypixel/hytale/server/core/event/`

**Key Events:**
- `PlayerConnectEvent` - Player joins
- `PlayerDisconnectEvent` - Player leaves
- `PlayerMouseButtonEvent` - Mouse input
- `PlayerMouseMotionEvent` - Mouse movement
- `ChangeGameModeEvent` - Game mode change
- `AddPlayerToWorldEvent` - Player enters world
- `DrainPlayerFromWorldEvent` - Player leaves world

### Event Priority

Events have priority levels:
- `FIRST` - Run first
- `EARLY` - Run early
- `NORMAL` - Default
- `LATE` - Run late
- `LAST` - Run last

**Side Effects:**
- Events can be cancelled
- Priority affects execution order
- Events enable plugin hooks

**Pattern Example:**
```java
// Event registration
getEventRegistry().register(PlayerMouseButtonEvent.class, event -> {
    Player player = event.getPlayer();
    MouseButtonType button = event.getMouseButton().mouseButtonType;
    Entity target = event.getTargetEntity();
    Item heldItem = event.getItemInHand();

    if (button == MouseButtonType.Right && target != null) {
        // Handle right-click on entity
    }
});

// With priority
getEventRegistry().register(PlayerConnectEvent.class, EventPriority.EARLY, event -> {
    // Handle early
});
```

---

## Additional Discoveries

### GameMode System
`#GameMode` `#Creative`

**Path:** `com/hypixel/hytale/protocol/GameMode.class`
**Path:** `com/hypixel/hytale/server/core/asset/type/gamemode/GameModeType.class`

GameMode likely includes:
- Adventure
- Creative
- Survival (?)

**Creative-specific:**
- `PlayerCreativeSettings` - Creative mode settings
- `SetCreativeItem` - Give creative items
- `DropCreativeItem` - Drop creative items

### NPC Spawning
`#NPC` `#Spawn`

**Path:** `com/hypixel/hytale/server/npc/`

NPC spawn via:
```java
int roleIndex = NPCPlugin.get().getIndex("Cow_Calf");
Pair<Ref<EntityStore>, NPCEntity> result = NPCPlugin.get().spawnEntity(
    store, roleIndex, position, rotation, null, null);
```

---

## NPC AI System
`#NPC` `#AI` `#Blackboard`

### Architecture Overview

The NPC AI uses a sophisticated **Blackboard + Action/Sensor** architecture:

**Key Packages:**
- `com.hypixel.hytale.server.npc.blackboard` - Shared data between AI components
- `com.hypixel.hytale.server.npc.corecomponents` - Actions and Sensors
- `com.hypixel.hytale.server.npc.role` - Role definitions (NPC templates)
- `com.hypixel.hytale.server.npc.instructions` - Instruction execution

### Blackboard Pattern

**Path:** `com.hypixel.hytale.server.npc.blackboard.Blackboard`

The Blackboard is a shared data store that AI components use to communicate:
- `AttitudeView` - Entity attitudes (friendly, hostile, neutral)
- `CombatViewSystems` - Combat state tracking
- `BlockTypeView` - Block awareness
- `EventView` - Event notifications
- `InteractionView` - Interaction state
- `ResourceView` - Resource reservations

```java
// Blackboard provides views for different AI needs
public <View extends IBlackboardView<View>> View getView(
    Class<View> viewClass,
    Ref<EntityStore> entityRef,
    ComponentAccessor<EntityStore> accessor);
```

### Role System

**Path:** `com.hypixel.hytale.server.npc.role.Role`

Roles define NPC templates with:
- `CombatSupport` - Combat capabilities
- `StateSupport` - State machine
- `EntitySupport` - Entity awareness
- `PositionCache` - Position/pathfinding cache
- `DebugSupport` - Debug tools

**Role Fields:**
```java
// Combat
protected final int initialMaxHealth;
protected final boolean invulnerable;
protected final double knockbackScale;

// Movement
protected final Steering bodySteering;
protected final Steering headSteering;
protected final SteeringForceAvoidCollision steeringForceAvoidCollision;

// Flocking
protected final double flockWeightAlignment;
protected final double flockWeightSeparation;
protected final double flockWeightCohesion;

// Inventory
protected final String[] hotbarItems;
protected final String[] offHandItems;
protected final String dropListId;
```

### Action System

**Path:** `com.hypixel.hytale.server.npc.corecomponents.ActionBase`

Actions are executable behaviors:

```java
public abstract class ActionBase {
    protected boolean once;      // Execute once only
    protected boolean triggered; // Has been triggered
    protected boolean active;    // Currently active

    boolean canExecute(entityRef, role, infoProvider, deltaTime, store);
    boolean execute(entityRef, role, infoProvider, deltaTime, store);
    void activate(role, infoProvider);
    void deactivate(role, infoProvider);
}
```

**Action Categories:**
- **Audiovisual:** `ActionPlayAnimation`, `ActionPlaySound`, `ActionSpawnParticles`
- **Combat:** `ActionAttack`, `ActionApplyEntityEffect`
- **Entity:** `ActionNotify`, `ActionSetStat`, `ActionOverrideAttitude`
- **Items:** `ActionPickUpItem`, `ActionDropItem`, `ActionInventory`
- **Lifecycle:** `ActionSpawn`, `ActionDespawn`, `ActionDie`, `ActionRole`
- **Movement:** `ActionCrouch`, `BodyMotionFind`, `BodyMotionMaintainDistance`
- **Interaction:** `ActionSetInteractable`, `ActionLockOnInteractionTarget`

### Sensor System

Sensors observe the world and provide data to actions:

**Entity Sensors:**
- `SensorEntity` - Detect entities
- `SensorPlayer` - Detect players specifically
- `SensorTarget` - Track current target
- `SensorBeacon` - Spawn beacon awareness
- `SensorKill` - Detect kills

**Combat Sensors:**
- `SensorDamage` - Damage detection
- `SensorIsBackingAway` - Retreat detection

**Entity Filters:**
- `EntityFilterAttitude` - Filter by attitude
- `EntityFilterLineOfSight` - Line of sight check
- `EntityFilterMovementState` - Movement state check
- `EntityFilterStat` - Stat-based filter
- `EntityFilterViewSector` - View cone filter

### Motion Controllers

NPCs use motion controllers for movement:
- `BodyMotionFind` - Pathfinding to target
- `BodyMotionMaintainDistance` - Keep distance from target
- `BodyMotionLand` - Landing behavior
- `BodyMotionLeave` - Leave area
- `HeadMotionWatch` - Watch target
- `HeadMotionAim` - Aim at target

---

## Combat & Damage System
`#Combat` `#Damage`

### Damage Architecture

**Path:** `com.hypixel.hytale.server.core.modules.entity.damage/`

**Key Classes:**
- `Damage` - Damage event object
- `DamageCause` - Type of damage (asset-based)
- `DamageSystems` - Processing systems
- `DamageCalculatorSystems` - Damage calculation

### Damage Class

```java
public class Damage extends CancellableEcsEvent {
    // Metadata
    public static final MetaKey<Vector4d> HIT_LOCATION;
    public static final MetaKey<Float> HIT_ANGLE;
    public static final MetaKey<Particles> IMPACT_PARTICLES;
    public static final MetaKey<SoundEffect> IMPACT_SOUND_EFFECT;
    public static final MetaKey<CameraEffect> CAMERA_EFFECT;
    public static final MetaKey<Boolean> BLOCKED;
    public static final MetaKey<KnockbackComponent> KNOCKBACK_COMPONENT;

    // Core fields
    private final float initialAmount;
    private int damageCauseIndex;
    private Damage.Source source;
    private float amount;
}
```

### Damage Sources

```java
// Source types in Damage class
Damage.EntitySource      // From entity (NPC, player)
Damage.ProjectileSource  // From projectile
Damage.EnvironmentSource // From environment (fire, lava)
Damage.CommandSource     // From command
```

### Damage Causes (Asset-Based)

**Path:** `com.hypixel.hytale.server.core.modules.entity.damage.DamageCause`

Pre-defined causes:
```java
PHYSICAL     // Melee attacks
PROJECTILE   // Ranged attacks
COMMAND      // Admin commands
DROWNING     // Water
ENVIRONMENT  // World hazards
FALL         // Fall damage
OUT_OF_WORLD // Void damage
SUFFOCATION  // Block suffocation
```

**Cause Properties:**
```java
protected boolean durabilityLoss;    // Damages equipment
protected boolean staminaLoss;       // Drains stamina
protected boolean bypassResistances; // Ignores armor
protected String animationId;        // Hit animation
protected String deathAnimationId;   // Death animation
```

### Damage Processing Systems

**Path:** `com.hypixel.hytale.server.core.modules.entity.damage.DamageSystems`

Processing pipeline:
1. `FilterPlayerWorldConfig` / `FilterNPCWorldConfig` - World rules
2. `FilterUnkillable` - Check invulnerability
3. `ArmorDamageReduction` - Apply armor resistance
4. `WieldingDamageReduction` - Wielding item reduction
5. `ArmorKnockbackReduction` - Knockback mitigation
6. `DamageStamina` - Stamina drain
7. `DamageArmor` - Armor durability loss
8. `ApplyDamage` - Apply final damage
9. `HitAnimation` - Play hit effects
10. `ApplyParticles` / `ApplySoundEffects` - Effects

**Specialized Systems:**
- `FallDamagePlayers` / `FallDamageNPCs` - Fall damage
- `OutOfWorldDamage` - Void damage
- `PlayerHitIndicators` - Hit feedback
- `RecordLastCombat` - Combat tracking

### Damage Calculation

```java
// Queue damage calculation
Damage[] damages = DamageCalculatorSystems.queueDamageCalculator(
    world,
    damageCausesMap,  // Object2FloatMap<DamageCause>
    targetRef,
    commandBuffer,
    damageSource,
    weaponItemStack
);
```

---

## Network Protocol
`#Net` `#Protocol`

### Packet Organization

**Path:** `com.hypixel.hytale.protocol.packets/`

**Packet Categories:**
- `asseteditor/` - Asset editor communication
- `assets/` - Asset updates
- `blocks/` - Block operations
- `entities/` - Entity sync
- `inventory/` - Inventory operations
- `player/` - Player state
- `worldmap/` - World map data

### Key Packets

**Player Packets:**
- `SetGameMode` - Game mode change
- `UpdatePlayerSettings` - Settings sync

**Entity Packets:**
- Entity spawn/despawn
- Position updates
- Animation triggers

**Inventory Packets:**
- `InventoryAction` - Inventory operations
- Container updates

**World Packets:**
- Block changes
- Chunk data
- Weather updates

---

## World Generation System
`#WorldGen` `#Terrain` `#Biome`

### Architecture Overview

The world generation system uses a hierarchical zone/biome/chunk architecture with procedural generation:

**Key Packages:**
- `com.hypixel.hytale.server.worldgen` - Main worldgen implementation
- `com.hypixel.hytale.server.core.universe.world.worldgen` - Core worldgen interfaces
- `com.hypixel.hytale.server.worldgen.biome` - Biome definitions
- `com.hypixel.hytale.server.worldgen.zone` - Zone system
- `com.hypixel.hytale.server.worldgen.cave` - Cave generation
- `com.hypixel.hytale.server.worldgen.climate` - Climate/temperature system
- `com.hypixel.hytale.server.worldgen.chunk` - Chunk generation

### World Generation Interface

**Path:** `universe.world.worldgen.IWorldGen`

```java
public interface IWorldGen {
    WorldGenTimingsCollector getTimings();
    CompletableFuture<GeneratedChunk> generate(int seed, long, int chunkX, int chunkZ, LongPredicate);
    Transform[] getSpawnPoints(int seed);
    ISpawnProvider getDefaultSpawnProvider(int seed);
    void shutdown();
}
```

### World Generation Providers

**Path:** `universe.world.worldgen.provider/`

| Provider | Purpose |
|----------|---------|
| `IWorldGenProvider` | Base interface for world generation |
| `FlatWorldGenProvider` | Flat world generation with layers |
| `VoidWorldGenProvider` | Empty void world |
| `DummyWorldGenProvider` | Placeholder generator |
| `HytaleWorldGenProvider` | Main Hytale terrain generation |

### Chunk Generator

**Path:** `server.worldgen.chunk.ChunkGenerator`

Main chunk generation orchestrator:

```java
public class ChunkGenerator implements IBenchmarkableWorldGen {
    // Core components
    private final ZonePatternProvider zonePatternProvider;
    private final ZonePatternGeneratorCache zonePatternGeneratorCache;
    private final ChunkGeneratorCache generatorCache;
    private final CaveGeneratorCache caveGeneratorCache;
    private final PrefabLoadingCache prefabLoadingCache;
    private final UniquePrefabCache uniquePrefabCache;

    // Generation methods
    CompletableFuture<GeneratedChunk> generate(seed, long, chunkX, chunkZ, predicate);
    ZoneBiomeResult getZoneBiomeResultAt(seed, x, z);
    int getHeight(seed, x, z);
    Cave getCave(CaveType, seed, x, z);
    Transform[] getSpawnPoints(seed);
}
```

### Zone System

**Path:** `server.worldgen.zone/`

Zones are large regions with distinct characteristics:

```java
public record Zone(
    int id,
    String name,
    ZoneDiscoveryConfig discoveryConfig,
    CaveGenerator caveGenerator,
    BiomePatternGenerator biomePatternGenerator,
    UniquePrefabContainer uniquePrefabContainer
) {}
```

| Component | Purpose |
|-----------|---------|
| `Zone` | Zone definition with biomes and caves |
| `ZonePatternProvider` | Provides zone patterns |
| `ZonePatternGenerator` | Generates zone patterns |
| `ZonePatternGeneratorCache` | Caches zone generation |
| `ZoneDiscoveryConfig` | Zone discovery settings |
| `ZoneColorMapping` | Zone map colors |

### Biome System

**Path:** `server.worldgen.biome/`

```java
public abstract class Biome {
    protected final int id;
    protected final String name;
    protected final BiomeInterpolation interpolation;
    protected final IHeightThresholdInterpreter heightmapInterpreter;
    protected final CoverContainer coverContainer;       // Surface cover
    protected final LayerContainer layerContainer;       // Terrain layers
    protected final PrefabContainer prefabContainer;     // Structure prefabs
    protected final TintContainer tintContainer;         // Grass/leaf colors
    protected final EnvironmentContainer environmentContainer;
    protected final WaterContainer waterContainer;       // Water settings
    protected final FadeContainer fadeContainer;         // Biome transitions
    protected final NoiseProperty heightmapNoise;        // Heightmap noise
    protected final int mapColor;                        // Map display color
}
```

**Biome Types:**
- `Biome` - Base biome class
- `CustomBiome` - Custom defined biomes
- `TileBiome` - Tile-based biomes
- `BiomePatternGenerator` - Generates biome patterns
- `BiomeInterpolation` - Biome edge blending

### Climate System

**Path:** `server.worldgen.climate/`

Climate determines biome placement:

```java
public class ClimateNoise {
    public final Grid grid;                    // Climate grid
    public final NoiseProperty continent;      // Continental noise
    public final NoiseProperty temperature;    // Temperature noise
    public final NoiseProperty intensity;      // Intensity noise
    public final Thresholds thresholds;        // Climate thresholds

    int generate(seed, x, z, Buffer, ClimateGraph);
}
```

| Component | Purpose |
|-----------|---------|
| `ClimateNoise` | Climate value generation |
| `ClimateGraph` | Climate → biome mapping |
| `ClimateType` | Climate classification |
| `ClimatePoint` | Climate data point |
| `ClimateSearch` | Climate-based searches |
| `UniqueClimateGenerator` | Unique climate features |

### Cave System

**Path:** `server.worldgen.cave/`

Procedural cave generation:

```java
public class CaveGenerator {
    private final CaveType[] caveTypes;

    Cave generate(seed, ChunkGenerator, CaveType, x, y, z);
    void startCave(seed, generator, cave, origin, random);
    void continueNode(seed, generator, cave, node, depth, random);
}
```

**Cave Components:**
- `Cave` - Cave instance
- `CaveType` - Cave type configuration
- `CaveNodeType` - Node types in caves
- `CaveNode` - Individual cave node
- `CavePrefab` - Cave prefab structures
- `CaveYawMode` - Cave direction modes

**Cave Shapes:**
- `EllipsoidCaveNodeShape` - Spherical caves
- `CylinderCaveNodeShape` - Tube caves
- `PipeCaveNodeShape` - Pipe-shaped caves
- `DistortedCaveNodeShape` - Irregular caves

### Prefab System

**Path:** `server.worldgen.prefab/`

Pre-built structure placement:

| Component | Purpose |
|-----------|---------|
| `PrefabCategory` | Prefab categorization |
| `PrefabLoadingCache` | Prefab caching |
| `PrefabPasteUtil` | Prefab placement |
| `PrefabPatternGenerator` | Prefab pattern generation |
| `BlockPlacementMask` | Block placement rules |
| `UniquePrefabGenerator` | Unique structure placement |
| `UniquePrefabConfiguration` | Unique prefab settings |

### Chunk Populators

**Path:** `server.worldgen.chunk.populator/`

Post-generation chunk decoration:

| Populator | Purpose |
|-----------|---------|
| `BlockPopulator` | Block decoration |
| `CavePopulator` | Cave carving |
| `PrefabPopulator` | Prefab placement |
| `WaterPopulator` | Water/fluid placement |

### Generated Chunk Structure

**Path:** `universe.world.worldgen.GeneratedChunk`

```java
public class GeneratedChunk {
    private final GeneratedBlockChunk generatedBlockChunk;      // Block data
    private final GeneratedBlockStateChunk generatedBlockStateChunk; // Block states
    private final GeneratedEntityChunk generatedEntityChunk;    // Entities
    private final Holder<ChunkStore>[] sections;                // Chunk sections

    Holder<ChunkStore> toWorldChunk(World world);
}
```

### Container System

**Path:** `server.worldgen.container/`

Biome feature containers:

| Container | Purpose |
|-----------|---------|
| `CoverContainer` | Surface coverage (grass, flowers) |
| `LayerContainer` | Terrain layers (dirt, stone) |
| `PrefabContainer` | Structure prefabs |
| `TintContainer` | Color tinting |
| `EnvironmentContainer` | Environment settings |
| `WaterContainer` | Water configuration |
| `FadeContainer` | Biome transition fading |
| `UniquePrefabContainer` | Unique landmarks |

### Caching System

**Path:** `server.worldgen.cache/`

Performance-critical caching:

| Cache | Purpose |
|-------|---------|
| `ChunkGeneratorCache` | Chunk generation cache |
| `CaveGeneratorCache` | Cave generation cache |
| `CoordinateCache` | Coordinate-based cache |
| `ExtendedCoordinateCache` | Extended coordinate cache |
| `UniquePrefabCache` | Unique prefab cache |
| `InterpolatedBiomeCountList` | Biome interpolation cache |
| `CoreDataCacheEntry` | Core worldgen data |

---

## Appendix: Key Class Paths

| Component | Full Path |
|-----------|-----------|
| Main | `com.hypixel.hytale.Main` |
| HytaleServer | `com.hypixel.hytale.server.core.HytaleServer` |
| Player | `com.hypixel.hytale.server.core.entity.entities.Player` |
| LivingEntity | `com.hypixel.hytale.server.core.entity.LivingEntity` |
| Inventory | `com.hypixel.hytale.server.core.inventory.Inventory` |
| MovementManager | `com.hypixel.hytale.server.core.entity.entities.player.movement.MovementManager` |
| CollisionModule | `com.hypixel.hytale.server.core.modules.collision.CollisionModule` |
| InteractionManager | `com.hypixel.hytale.server.core.entity.InteractionManager` |
| InteractionType | `com.hypixel.hytale.protocol.InteractionType` |
| RaycastSelector | `com.hypixel.hytale.server.core.modules.interaction.interaction.config.selector.RaycastSelector` |
| PlayerWorldData | `com.hypixel.hytale.server.core.entity.entities.player.data.PlayerWorldData` |
| ISpawnProvider | `com.hypixel.hytale.server.core.universe.world.spawn.ISpawnProvider` |
| InstancesPlugin | `com.hypixel.hytale.builtin.instances.InstancesPlugin` |
