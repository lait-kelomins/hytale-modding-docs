# Hytale Server API - Questions & Answers

**Date:** 2026-01-19
**Source:** Reverse engineering of `HytaleServer.jar`

---

## Table of Contents

1. [No-Clip Mode / Creative Mode Toggle](#q1-no-clip-mode--creative-mode-toggle)
2. [Player Flight](#q2-player-flight)
3. [Disable Player Collisions](#q3-disable-player-collisions)
4. [Change Default Player Spawn](#q4-change-default-player-spawn)
5. [Hotkey System and Interactions](#q5-hotkey-system-and-interactions)
6. [Spawning Algorithm](#q6-spawning-algorithm)
7. [Get Block Player is Looking At](#q7-get-block-player-is-looking-at)
8. [Separate Inventories Between Worlds](#q8-separate-inventories-between-worlds)
9. [Separate Inventories Between Instances](#q9-separate-inventories-between-instances)
10. [Player Interaction Types](#q10-player-interaction-types)

---

## Q1: No-Clip Mode / Creative Mode Toggle

### Question
How to set the player in no-clip mode? How does the game do it in the creative mode inventory UI toggle?

### Answer

**Game Modes Available:**
The game has only 2 game modes defined in `com.hypixel.hytale.protocol.GameMode`:
- `Adventure` - Normal gameplay
- `Creative` - Creative mode with flight

**Key Classes:**
- `GameMode` enum: `com.hypixel.hytale.protocol.GameMode`
- `PlayerCreativeSettings`: `com.hypixel.hytale.server.core.modules.entity.player.PlayerCreativeSettings`
- `MovementSettings`: `com.hypixel.hytale.protocol.MovementSettings`

**How Creative Mode Works:**

1. **GameMode Field**: The `Player` class has `private com.hypixel.hytale.protocol.GameMode gameMode;`

2. **MovementSettings.canFly**: The `MovementSettings` protocol class has a `boolean canFly` field that enables flight capability.

3. **PlayerCreativeSettings**: A simple record with:
   - `allowNPCDetection` - Whether NPCs detect the player (set to false = invisible to mobs)
   - `respondToHit` - Whether player takes damage (set to false = invulnerable)

**No-Clip Implementation Pattern:**

No-clip appears to be achieved through a combination of:
1. Setting `canFly = true` in MovementSettings
2. Potentially disabling collision (see Q3)
3. Setting Creative game mode which enables flight

```java
// Conceptual pattern for enabling creative/no-clip
Player player = getPlayer();

// Change game mode
player.setGameMode(GameMode.Creative);

// The MovementManager's setDefaultSettings is called with GameMode
// which sets canFly based on the game mode
MovementManager movementManager = getMovementManager(player);
MovementSettings settings = movementManager.getSettings();
settings.canFly = true;
movementManager.update(packetHandler);

// Creative settings for invulnerability
PlayerCreativeSettings creativeSettings = new PlayerCreativeSettings(
    false,  // allowNPCDetection - false = invisible to NPCs
    false   // respondToHit - false = invulnerable
);
```

**Packet to Send:**
`com.hypixel.hytale.protocol.packets.player.SetGameMode`

---

## Q2: Player Flight

### Question
How to make the player fly?

### Answer

**Key Classes:**
- `MovementSettings`: `com.hypixel.hytale.protocol.MovementSettings`
- `MovementStates`: `com.hypixel.hytale.protocol.MovementStates`
- `MovementManager`: `com.hypixel.hytale.server.core.entity.entities.player.movement.MovementManager`
- `MovementConfig`: Asset-driven configuration in `Server/Entity/MovementConfig/`

**Flight Control Fields in MovementSettings:**
```java
public boolean canFly;              // CRITICAL: Enables/disables flight capability
public float horizontalFlySpeed;    // Default: 10.32
public float verticalFlySpeed;      // Default: 10.32
```

**Movement States (from MovementStates class):**
```java
public boolean flying;      // Currently flying
public boolean gliding;     // Using glider
public boolean falling;     // Falling
public boolean onGround;    // On ground
```

**Implementation Pattern:**
```java
// Get the player's MovementManager component
MovementManager movementManager = store.getComponent(playerRef,
    MovementManager.getComponentType());

// Get current settings
MovementSettings settings = movementManager.getSettings();

// Enable flight
settings.canFly = true;

// Optionally adjust fly speed
settings.horizontalFlySpeed = 15.0f;  // Faster horizontal
settings.verticalFlySpeed = 15.0f;    // Faster vertical

// Send update to client
movementManager.update(player.getPlayerConnection());
```

**Default MovementConfig Values (from Default.json):**
```json
{
  "HorizontalFlySpeed": 10.32,
  "VerticalFlySpeed": 10.32
}
```

**Note:** Flight is automatically enabled when game mode is `Creative`. The `MovementManager.setDefaultSettings()` method takes `GameMode` as a parameter and configures `canFly` accordingly.

---

## Q3: Disable Player Collisions

### Question
How to disable all collisions for a specific player?

### Answer

**Key Classes:**
- `CollisionConfig`: `com.hypixel.hytale.server.core.modules.collision.CollisionConfig`
- `CollisionFilter`: `com.hypixel.hytale.server.core.modules.collision.CollisionFilter`
- `CollisionModule`: `com.hypixel.hytale.server.core.modules.collision.CollisionModule`
- `CharacterCollisionData`: `com.hypixel.hytale.server.core.modules.collision.CharacterCollisionData`

**CollisionConfig Key Fields:**
```java
public boolean blockCanCollide;        // Whether blocks can collide
public boolean blockCanTrigger;        // Whether blocks can trigger
public boolean blockCanTriggerPartial; // Partial trigger
public boolean checkTriggerBlocks;     // Check trigger blocks
public boolean checkDamageBlocks;      // Check damage blocks
public Predicate<CollisionConfig> canCollide;  // Custom collision predicate
```

**Collision Materials (constants in CollisionConfig):**
```java
MATERIAL_EMPTY = 0
MATERIAL_FLUID = 1
MATERIAL_SOLID = 2
MATERIAL_SUBMERGED = 3
MATERIAL_DAMAGE = 4
MATERIAL_SET_NONE = 0
MATERIAL_SET_ANY = -1
```

**Methods Available:**
```java
void setCollisionByMaterial(int materialMask)
void setDefaultCollisionBehaviour()
void setDefaultBlockCollisionPredicate()
boolean isCheckTriggerBlocks()
void setCheckTriggerBlocks(boolean)
boolean isCheckDamageBlocks()
void setCheckDamageBlocks(boolean)
```

**Implementation Pattern:**
```java
// Option 1: Use CollisionConfig to disable block collision
CollisionConfig config = getCollisionConfig(player);
config.blockCanCollide = false;
config.blockCanTrigger = false;
config.setCollisionByMaterial(CollisionConfig.MATERIAL_SET_NONE);

// Option 2: Use custom predicate that always returns false
config.canCollide = (cfg) -> false;

// Option 3: Use EntityIntangibleCommand pattern
// The game has EntityIntangibleCommand which likely sets intangibility
// Look at: com.hypixel.hytale.server.core.command.commands.world.entity.EntityIntangibleCommand
```

**Debug Commands Available:**
- `HitboxCollisionAddCommand` - Adds hitbox collision
- `HitboxCollisionRemoveCommand` - Removes hitbox collision

These are in `server/core/command/commands/debug/component/hitboxcollision/`

---

## Q4: Change Default Player Spawn

### Question
How to change the default spawn of a player?

### Answer

**Key Classes:**
- `ISpawnProvider`: `com.hypixel.hytale.server.core.universe.world.spawn.ISpawnProvider`
- `GlobalSpawnProvider`: Single spawn point for all players
- `IndividualSpawnProvider`: Per-player spawn points
- `FitToHeightMapSpawnProvider`: Spawn at terrain height
- `WorldSpawnPoint`: `com.hypixel.hytale.server.core.asset.type.gameplay.respawn.WorldSpawnPoint`
- `SpawnConfig`: `com.hypixel.hytale.server.core.asset.type.gameplay.SpawnConfig`
- `PlayerRespawnPointData`: `com.hypixel.hytale.server.core.entity.entities.player.data.PlayerRespawnPointData`

**ISpawnProvider Interface:**
```java
public interface ISpawnProvider {
    Transform getSpawnPoint(World world, UUID playerId);
    Transform[] getSpawnPoints();
    boolean isWithinSpawnDistance(Vector3d position, double distance);
}
```

**Implementation Options:**

### Option 1: World-Level Spawn (GlobalSpawnProvider)
```java
// The world has a spawn provider configured
// Use WorldConfigSetSpawnCommand pattern
// com.hypixel.hytale.server.core.universe.world.commands.worldconfig.WorldConfigSetSpawnCommand

World world = player.getWorld();
// Set world spawn point through WorldConfig
```

### Option 2: Per-Player Spawn (IndividualSpawnProvider)
```java
// Store individual spawn points per player UUID
IndividualSpawnProvider provider = new IndividualSpawnProvider();
provider.setSpawnPoint(playerId, new Transform(x, y, z, rotation));
```

### Option 3: Player's Respawn Point Data
```java
// Each player has respawn point data stored per-world
Player player = getPlayer();
PlayerWorldData worldData = player.getPlayerConfigData().getWorldData(worldId);
PlayerRespawnPointData[] respawnPoints = worldData.getRespawnPoints();
// Modify respawn points array
worldData.setRespawnPoints(newRespawnPoints);
```

**Commands Available:**
- `WorldConfigSetSpawnCommand` - Sets world spawn
- `WorldConfigSetSpawnDefaultCommand` - Resets to default

**Asset Configuration:**
Spawn configuration can be defined in asset files:
- `Server/Gameplay/SpawnConfig.json` (hypothetical path)
- World-specific configs

---

## Q5: Hotkey System and Interactions

### Question
How to add a hotkey and use that hotkey for interactions? Where are Use, Primary, Secondary defined?

### Answer

**Key Discovery: InteractionType Enum**

**Path:** `com.hypixel.hytale.protocol.InteractionType`

The game defines interaction types (hotkeys) in an enum:

```java
public enum InteractionType {
    Primary,           // Left-click / Attack
    Secondary,         // Right-click alternate
    Ability1,          // Ability slot 1
    Ability2,          // Ability slot 2
    Ability3,          // Ability slot 3
    Use,               // E key / Interact
    Pick,              // Pick block (like Minecraft middle-click)
    Pickup,            // Item pickup
    CollisionEnter,    // Triggered on collision start
    CollisionLeave,    // Triggered on collision end
    Collision,         // During collision
    EntityStatEffect,  // Stat effect trigger
    SwapTo,            // Swapped to this item
    SwapFrom,          // Swapped from this item
    Death,             // On death trigger
    Wielding,          // While wielding
    ProjectileSpawn,   // Projectile created
    ProjectileHit,     // Projectile hit something
    ProjectileMiss,    // Projectile missed
    ProjectileBounce,  // Projectile bounced
    Held,              // Item held in main hand
    HeldOffhand,       // Item held in offhand
    Equipped,          // Item equipped
    Dodge,             // Dodge action
    GameModeSwap       // Game mode changed
}
```

**Key Classes:**
- `InteractionType`: Protocol enum defining all interaction types
- `InteractionTypeUtils`: `com.hypixel.hytale.server.core.modules.interaction.interaction.config.InteractionTypeUtils`
- `InteractionManager`: `com.hypixel.hytale.server.core.entity.InteractionManager`
- `InteractionChain`: Chains multiple interactions together
- `InteractionContext`: Context for current interaction

**Using Interactions:**

### 1. Registering via PlayerMouseButtonEvent
```java
getEventRegistry().register(PlayerMouseButtonEvent.class, event -> {
    Player player = event.getPlayer();
    MouseButtonType button = event.getMouseButton().mouseButtonType;
    Entity target = event.getTargetEntity();
    Item heldItem = event.getItemInHand();

    // Primary = Left click, Secondary/Use = Right click
    if (button == MouseButtonType.Left) {
        // Primary action
    } else if (button == MouseButtonType.Right) {
        // Use/Secondary action
    }
});
```

### 2. Setting Entity Interactions
```java
// Set an interaction on an entity
Object interactions = store.ensureAndGetComponent(entityRef, interactionsCompType);

Class<?> interactionTypeClass = Class.forName("com.hypixel.hytale.protocol.InteractionType");
Object useType = Arrays.stream(interactionTypeClass.getEnumConstants())
    .filter(e -> e.toString().equals("Use"))
    .findFirst().orElse(null);

Method setIntId = interactions.getClass().getMethod("setInteractionId",
    interactionTypeClass, String.class);
setIntId.invoke(interactions, useType, "Root_MyCustomInteraction");
```

### 3. Asset-Based Interaction Definition
Interactions are defined in JSON assets:
- `Server/Item/RootInteractions/{id}.json` - Root interaction definitions
- `Server/Item/Interactions/{id}.json` - Interaction chains

**Can You Add New Hotkeys?**
The `InteractionType` enum is compiled into the protocol. Adding new types would require:
1. Modifying the client (not possible without official support)
2. Using existing types creatively (Ability1, Ability2, Ability3 are likely unused slots)

**Recommendation:** Use the existing `Ability1`, `Ability2`, `Ability3` types for custom hotkey bindings.

---

## Q6: Spawning Algorithm

### Question
How does spawning work? What are the algorithm, conditions, config? Can it be changed?

### Answer

**Spawning Systems Identified:**

### 1. Block Spawner Plugin
**Path:** `com.hypixel.hytale.builtin.blockspawner.BlockSpawnerPlugin`

Block-based spawning system (like Minecraft's mob spawners):
- `BlockSpawnerEntry` - Entry with rotation modes
- `BlockSpawnerTable` - Table of possible spawns
- `BlockSpawner` - State component stored per-chunk

### 2. NPC Spawn System
**Path:** `com.hypixel.hytale.server.npc.`

NPC spawning with markers and beacons:
- Spawn Markers: `Server/NPC/Spawn/Markers/`
- Spawn Beacons: `Server/NPC/Spawn/Beacons/`
- Spawn Suppression: `Server/NPC/Spawn/Suppression/`
- World Spawns: `Server/NPC/Spawn/World/`

### 3. Prefab Spawner
**Path:** `com.hypixel.hytale.server.core.modules.prefabspawner.`

Spawns prefabs (structures/entities):
- `PrefabSpawnerModule`
- `PrefabSpawnerState`

**Spawn Configuration Assets:**

**Zone-Based Spawns:**
```
Server/NPC/Spawn/World/Zone1/Spawns_Zone1_*.json
Server/NPC/Spawn/World/Zone2/Spawns_Zone2_*.json
Server/NPC/Spawn/World/Zone3/Spawns_Zone3_*.json
```

**Spawn Suppression:**
```
Server/NPC/Spawn/Suppression/Spawn_Camp.json
Server/Instances/*/resources/SpawnSuppressionController.json
```

**NPC Spawning Pattern:**
```java
// Using NPCPlugin to spawn
int roleIndex = NPCPlugin.get().getIndex("Cow_Calf");
Pair<Ref<EntityStore>, NPCEntity> result = NPCPlugin.get().spawnEntity(
    store, roleIndex, position, rotation, null, null);
```

**Can It Be Changed?**
Yes, spawning can be customized through:
1. Asset pack modifications (JSON files)
2. Plugin event listeners
3. Custom spawn providers implementing `ISpawnProvider`
4. Block spawner configuration
5. Spawn suppression zones

---

## Q7: Get Block Player is Looking At

### Question
How can I get the block the player is looking at?

### Answer

**Key Classes:**
- `RaycastSelector`: `com.hypixel.hytale.server.core.modules.interaction.interaction.config.selector.RaycastSelector`
- `RaycastAABB`: `com.hypixel.hytale.math.raycast.RaycastAABB`
- `RaycastMode`: `com.hypixel.hytale.protocol.RaycastMode`

**RaycastSelector Fields:**
```java
protected Vector3d offset;                    // Position offset
protected int distance;                       // Max raycast distance
protected boolean ignoreFluids;               // Skip fluid blocks
protected boolean ignoreEmptyCollisionMaterial;  // Skip non-solid
protected String blockTag;                    // Filter by block tag
protected int blockTagIndex;                  // Tag index
```

**RaycastSelector Methods:**
```java
Vector3d selectTargetPosition(CommandBuffer<EntityStore> buffer,
                              Ref<EntityStore> entityRef);
Selector newSelector();
```

**Implementation Pattern:**
```java
// Option 1: Using RaycastSelector
RaycastSelector selector = new RaycastSelector();
selector.distance = 10;  // Max distance in blocks
selector.ignoreFluids = true;
selector.ignoreEmptyCollisionMaterial = true;

Vector3d targetPosition = selector.selectTargetPosition(commandBuffer, playerRef);
// targetPosition contains the block position player is looking at

// Option 2: Using RaycastAABB directly
Vector3d eyePos = player.getEyePosition();
Vector3d direction = player.getLookDirection();
float maxDistance = 10.0f;

RaycastAABB.raycast(eyePos, direction, maxDistance, (hitX, hitY, hitZ, data) -> {
    // Called for each block hit
    // Return true to continue, false to stop
    BlockPosition lookingAt = new BlockPosition(hitX, hitY, hitZ);
    return false; // Stop at first hit
});
```

**Available in Event:**
The `PlayerMouseButtonEvent` may include target information:
```java
getEventRegistry().register(PlayerMouseButtonEvent.class, event -> {
    Entity targetEntity = event.getTargetEntity();  // Entity being looked at
    // Block target may be available through event context
});
```

---

## Q8: Separate Inventories Between Worlds

### Question
Is there any way to separate inventories between worlds in the same server?

### Answer

**Key Discovery: PlayerWorldData**

**Path:** `com.hypixel.hytale.server.core.entity.entities.player.data.PlayerWorldData`

The game already has a per-world data system:

```java
public final class PlayerWorldData {
    private PlayerConfigData playerConfigData;
    private Transform lastPosition;            // Last position in this world
    private SavedMovementStates lastMovementStates;
    private MapMarker[] worldMapMarkers;       // World map markers
    private boolean firstSpawn;                // First spawn in world
    private PlayerRespawnPointData[] respawnPoints;  // Respawn points
    private List<PlayerDeathPositionData> deathPositions;  // Death locations
}
```

**Architecture:**
- `PlayerConfigData` - Root player configuration
- `PlayerWorldData` - Per-world player data
- Each world can have separate `PlayerWorldData`

**Implementation Approach:**

### Option 1: Use Existing PlayerWorldData
The `PlayerWorldData` is already per-world. You would need to:
1. Store inventory data within or alongside `PlayerWorldData`
2. Save/restore inventory when player changes worlds

```java
// Events to hook:
AddPlayerToWorldEvent     // Player enters a world
DrainPlayerFromWorldEvent // Player leaves a world

// Pattern:
getEventRegistry().register(AddPlayerToWorldEvent.class, event -> {
    Player player = event.getPlayer();
    World world = event.getWorld();
    String worldId = world.getId();

    // Load world-specific inventory
    Inventory worldInventory = loadInventoryForWorld(player.getUUID(), worldId);
    player.setInventory(worldInventory);
});

getEventRegistry().register(DrainPlayerFromWorldEvent.class, event -> {
    Player player = event.getPlayer();
    World previousWorld = event.getWorld();

    // Save current inventory for this world
    saveInventoryForWorld(player.getUUID(), previousWorld.getId(), player.getInventory());
});
```

### Option 2: Plugin-Based Inventory Management
Create a plugin that:
1. Listens to world transfer events
2. Serializes inventory on world exit
3. Deserializes inventory on world entry
4. Stores data in a database or files

**Limitation:** The base `Inventory` class is on `LivingEntity`, not `PlayerWorldData`. Custom storage/management is required.

---

## Q9: Separate Inventories Between Instances

### Question
Is there any way to separate inventories between instances in the same server?

### Answer

**Key Classes:**
- `InstancesPlugin`: `com.hypixel.hytale.builtin.instances.InstancesPlugin`
- `InstanceEntityConfig`: `com.hypixel.hytale.builtin.instances.config.InstanceEntityConfig`
- `InstanceWorldConfig`: `com.hypixel.hytale.builtin.instances.config.InstanceWorldConfig`
- `WorldReturnPoint`: `com.hypixel.hytale.builtin.instances.config.WorldReturnPoint`

**InstanceEntityConfig:**
```java
public class InstanceEntityConfig implements Component<EntityStore> {
    private WorldReturnPoint returnPoint;          // Where to return after instance
    private transient WorldReturnPoint returnPointOverride;  // Override point
}
```

**Instance System Architecture:**
- Instances are separate worlds with their own configurations
- `InstanceEntityConfig` is a component attached to entities (players) in instances
- `WorldReturnPoint` stores where to return the player

**Instance Features:**
- Timeout-based cleanup (`TimeoutCondition`, `IdleTimeoutCondition`)
- World empty detection (`WorldEmptyCondition`)
- Exit interactions (`ExitInstanceInteraction`)

**Implementation for Instance-Separate Inventories:**

Since instances are technically separate worlds, you can use the same approach as Q8:

```java
// Check if world is an instance
InstanceWorldConfig instanceConfig = getInstanceWorldConfig(world);
if (instanceConfig != null) {
    // This is an instance world
    // Apply instance-specific inventory rules
}

// Events for instance transitions:
// TeleportInstanceInteraction triggers instance entry
// ExitInstanceInteraction triggers instance exit

// Pattern for instance inventory:
getEventRegistry().register(AddPlayerToWorldEvent.class, event -> {
    World world = event.getWorld();
    Player player = event.getPlayer();

    InstanceWorldConfig instanceConfig = getInstanceConfig(world);
    if (instanceConfig != null) {
        // Player entered an instance
        // Option 1: Clear inventory for fresh start
        player.getInventory().clear();

        // Option 2: Load instance-specific inventory
        Inventory instanceInv = loadInstanceInventory(player, instanceConfig);
        player.setInventory(instanceInv);

        // Option 3: Preserve main world inventory (store separately)
        saveMainWorldInventory(player);
        player.getInventory().clear();
    }
});
```

**Instance Configuration Locations:**
```
Server/Instances/Default/
Server/Instances/Dungeon_1/
Server/Instances/Persistent/
etc.
```

---

## Q10: Player Interaction Types

### Question
What type of player interactions are there and which ones are used and seem to reliably work in the code?

### Answer

**Complete InteractionType Enum:**

| Type | Value | Description | Reliability |
|------|-------|-------------|-------------|
| `Primary` | 0 | Left-click / Attack | High - Core mechanic |
| `Secondary` | 1 | Right-click alternate | High - Core mechanic |
| `Ability1` | 2 | Ability slot 1 | Medium - May be unused |
| `Ability2` | 3 | Ability slot 2 | Medium - May be unused |
| `Ability3` | 4 | Ability slot 3 | Medium - May be unused |
| `Use` | 5 | E key / Interact | High - Core mechanic |
| `Pick` | 6 | Pick block | High - Creative mode |
| `Pickup` | 7 | Item pickup | High - Automatic |
| `CollisionEnter` | 8 | Collision start | High - Physics |
| `CollisionLeave` | 9 | Collision end | High - Physics |
| `Collision` | 10 | During collision | High - Physics |
| `EntityStatEffect` | 11 | Stat effect | Medium |
| `SwapTo` | 12 | Item swap in | Medium |
| `SwapFrom` | 13 | Item swap out | Medium |
| `Death` | 14 | On death | High - Core mechanic |
| `Wielding` | 15 | While wielding | High - Combat |
| `ProjectileSpawn` | 16 | Projectile created | High - Combat |
| `ProjectileHit` | 17 | Projectile hit | High - Combat |
| `ProjectileMiss` | 18 | Projectile miss | Medium |
| `ProjectileBounce` | 19 | Projectile bounce | Medium |
| `Held` | 20 | Main hand held | High |
| `HeldOffhand` | 21 | Offhand held | High |
| `Equipped` | 22 | Item equipped | High |
| `Dodge` | 23 | Dodge action | Medium - Advanced |
| `GameModeSwap` | 24 | Game mode change | High |

**Most Reliable for Plugins:**

1. **`Use`** - Best for custom entity/block interactions (Right-click / E key)
2. **`Primary`** - Reliable for attack-based mechanics (Left-click)
3. **`CollisionEnter/Leave`** - Good for trigger zones
4. **`Death`** - Reliable for death-related mechanics
5. **`ProjectileSpawn/Hit`** - Good for custom projectile systems

**Interaction Configuration Files:**
- `Server/Item/RootInteractions/` - Root interactions (entry points)
- `Server/Item/Interactions/` - Interaction chains

**Example Working Interactions from Assets:**
```
Root_FeedAnimal
Root_SpawnNPC
Root_NPC_Spawn_Void_Attack
SpawnNPC_Default
UseBlockInteraction
UseEntityInteraction
```

**PlayerMouseButtonEvent Integration:**
```java
getEventRegistry().register(PlayerMouseButtonEvent.class, event -> {
    MouseButtonType button = event.getMouseButton().mouseButtonType;

    // button corresponds to Primary (Left) or Use (Right)
    if (button == MouseButtonType.Left) {
        // Primary interaction
    } else if (button == MouseButtonType.Right) {
        // Use interaction (also Secondary for some items)
    }

    // Additional context:
    Entity target = event.getTargetEntity();  // What player clicked on
    Item heldItem = event.getItemInHand();    // Current held item
});
```

---

## Summary Table

| Question | Key Classes | Feasibility |
|----------|-------------|-------------|
| No-clip mode | `MovementSettings.canFly`, `GameMode` | High |
| Flight | `MovementSettings`, `MovementManager` | High |
| Disable collision | `CollisionConfig`, `CollisionFilter` | High |
| Change spawn | `ISpawnProvider`, `PlayerRespawnPointData` | High |
| Hotkeys | `InteractionType` enum | Medium (use existing) |
| Spawning | `BlockSpawnerPlugin`, `NPCPlugin` | High |
| Block look-at | `RaycastSelector`, `RaycastAABB` | High |
| World inventory | `PlayerWorldData`, events | Medium (custom impl) |
| Instance inventory | `InstanceEntityConfig`, events | Medium (custom impl) |
| Interaction types | `InteractionType` (26 types) | High |

---

## Appendix: Key Package Paths

```
Player Systems:     com.hypixel.hytale.server.core.entity.entities.player
Movement:           com.hypixel.hytale.server.core.entity.entities.player.movement
Collision:          com.hypixel.hytale.server.core.modules.collision
Interaction:        com.hypixel.hytale.server.core.modules.interaction
Spawn:              com.hypixel.hytale.server.core.universe.world.spawn
Inventory:          com.hypixel.hytale.server.core.inventory
Instance Plugin:    com.hypixel.hytale.builtin.instances
Protocol:           com.hypixel.hytale.protocol
```
