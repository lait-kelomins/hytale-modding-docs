# API Discoveries (Runtime Exploration)

These are actual API findings from runtime reflection, which differ from official docs.

## CommandContext

Located at: `com.hypixel.hytale.server.core.command.system.CommandContext`

**Methods:**
- `sender()` → CommandSender - Get the command sender
- `senderAsPlayerRef()` → Ref - Get player as entity reference
- `isPlayer()` → boolean - Check if sender is a player
- `sendMessage(Message)` → void - Send message to sender
- `getInputString()` → String - Raw input string
- `getCalledCommand()` → AbstractCommand - The command being executed
- `get(argName)` → Object - Get argument value
- `provided(argName)` → boolean - Check if argument was provided
- `getInput(argName)` → String[] - Get raw input for argument

**Fields:**
- `sender` : CommandSender
- `calledCommand` : AbstractCommand
- `inputString` : String
- `argValues` : Map
- `argInput` : Map

## HytaleServer

Located at: `com.hypixel.hytale.server.core.HytaleServer`

**Methods:**
- `get()` → HytaleServer (static) - Get server instance
- `getConfig()` → HytaleServerConfig
- `getServerName()` → String
- `getEventBus()` → EventBus
- `getPluginManager()` → PluginManager
- `getCommandManager()` → CommandManager
- `getShutdownReason()` → ShutdownReason
- `getBootStart()` → long
- `getBoot()` → Instant

**Note:** No direct `getUniverseManager()` or `getWorldManager()` visible in public methods.

## Interaction System

### Package Structure (actual)
- `SimpleInstantInteraction`: `com.hypixel.hytale.server.core.modules.interaction.interaction.config.SimpleInstantInteraction`
- `SimpleInteraction`: `com.hypixel.hytale.server.core.modules.interaction.interaction.config.SimpleInteraction`
- `Interaction`: `com.hypixel.hytale.server.core.modules.interaction.interaction.config.Interaction`
- `InteractionContext`: `com.hypixel.hytale.server.core.entity.InteractionContext`
- `InteractionType`: `com.hypixel.hytale.protocol.InteractionType`
- `InteractionState`: `com.hypixel.hytale.protocol.InteractionState`
- `CooldownHandler`: `com.hypixel.hytale.server.core.modules.interaction.interaction.CooldownHandler`
- `Interactions` (component): `com.hypixel.hytale.server.core.modules.interaction.Interactions`

### InteractionType (protocol)
Values: Primary (LMB), Secondary (RMB), Use (E), Scroll

### InteractionState (protocol)
Values: Finished, Skip, ItemChanged, Failed, NotFinished

### InteractionContext Methods
- `getEntity()` → Ref<EntityStore> - The entity performing interaction
- `getTargetEntity()` → Ref<EntityStore> - The target entity
- `getHeldItem()` → ItemStack - Currently held item
- `getTargetBlock()` → BlockPosition
- And many more...

## ItemStack

Located at: `com.hypixel.hytale.server.core.inventory.ItemStack`

**Methods:**
- `getItemId()` → String - NOT `getType().getId()`
- `getItem()` → Item
- `getQuantity()` → int
- `isEmpty()` → boolean
- `getMetadata()` → BsonDocument

## World/Universe

### Package Structure (actual)
- `Universe`: `com.hypixel.hytale.server.core.universe.Universe`
- `World`: `com.hypixel.hytale.server.core.universe.world.World`
- `EntityStore`: `com.hypixel.hytale.server.core.universe.world.storage.EntityStore`

### Accessing World from Player
```java
// ctx.sender() returns Player directly (not via senderAsPlayerRef)
Object player = ctx.sender();  // com.hypixel.hytale.server.core.entity.entities.Player
Object world = player.getWorld();  // World
Object entityStore = world.getEntityStore();  // EntityStore
```

### Player (com.hypixel.hytale.server.core.entity.entities.Player)
**Methods:**
- `getWorld()` → World
- `getWorldMapTracker()` → WorldMapTracker
- `getMountEntityId()` → int
- `unloadFromWorld()` → void

### EntityStore (wrapper)
**Methods:**
- `getStore()` → Store - The inner ECS store
- `getWorld()` → World
- `getRefFromNetworkId(int)` → Ref
- `getRefFromUUID(UUID)` → Ref

### Store (inner ECS store) - 285 entities
**Methods:**
- `getEntityCount()` → int - Total entity count (285 in test)
- `getRegistry()` → ComponentRegistry
- `getArchetype(Ref)` → Archetype
- `getExternalData()` → Object
- `getResourceStorage()` → IResourceStorage
- `getResource(ResourceType)` → Resource
- `getStoreIndex()` → int
- `getSystemMetrics()` → HistoricMetric[]
- `getFetchTask()` → ParallelTask
- `getParallelTask()` → ParallelTask
- `getArchetypeChunkCount()` → int
- `getArchetypeChunkCountFor(int)` → int
- `getEntityCountFor(Query)` → int
- `getEntityCountFor(int)` → int
- `copyEntity(Ref)` → Holder
- `copySerializableEntity(Ref)` → Holder
- `collectArchetypeChunkData()` → ArchetypeChunkData[] - **USEFUL: Returns array we can iterate!**

**Iteration methods (require specific thread):**
- `forEachChunk(BiPredicate)` → boolean
- `forEachChunk(BiConsumer)` → void
- `forEachEntityParallel(IntBiObjectConsumer)` → void - **Requires WorldThread!**

## Entity References

The ECS system uses `Ref<EntityStore>` instead of direct Entity references.
- `Ref.get()` retrieves the actual entity (but may return null!)
- `Ref.getStore()` retrieves the Store
- `Ref.getIndex()` → int - Entity index
- `Ref.isValid()` → boolean
- `Ref.validate()` → void
- Components are accessed through the entity store

### PlayerRef (from ctx.senderAsPlayerRef())
**Methods:**
- `toString()` → String
- `hashCode()` → Integer
- `validate()` → null (void)
- `getIndex()` → Integer
- `isValid()` → Boolean
- `getStore()` → Store
- `hashCode0()` → Integer
- `get()` → null (doesn't work for player entity access!)

**Note:** Use `ctx.sender()` directly to get the Player object, not `senderAsPlayerRef().get()`.

## Events

PlayerInteractEvent is **deprecated**. Alternatives:
- PlayerMouseButtonEvent - Only fires with unlocked mouse (menus)
- UseBlockEvent.Pre/Post - For block interactions only

For entity interactions during gameplay, use the Interaction system (attach interactions to entity's Interactions component).

---

## ECS Component System

### Archetype

Located at: `com.hypixel.hytale.component.Archetype`

**Fields:**
- `EMPTY` : Archetype - Empty archetype singleton
- `minIndex` : int - Minimum entity index
- `count` : int - Entity count
- `componentTypes` : ComponentType[] - Sparse array (len=132), indices are ComponentType IDs
- `exactQuery` : ExactArchetypeQuery

**Methods:**
- `length()` → int
- `count()` → int
- `isEmpty()` → boolean
- `getMinIndex()` → int
- `asExactQuery()` → ExactArchetypeQuery
- `empty()` → Archetype
- `validate()` → void
- `toString()` → String - Shows component class names

### ArchetypeChunk

**Methods:**
- `getComponent(int entityIndex, ComponentType)` → Component - Get component for entity at index
- `setComponent(int entityIndex, ComponentType, Component)` → void
- `getArchetype()` → Archetype
- `getReferenceTo(int entityIndex)` → Ref - Get entity reference at index
- `size()` → int - Number of entities in this chunk

### ComponentType Registry (sparse array indices)

Entity components are stored at specific indices in a sparse array:

| Index | Component Class | Package |
|-------|----------------|---------|
| 11 | BoundingBox | `com.hypixel.hytale.server.core.modules.entity.component` |
| 13 | TransformComponent | `com.hypixel.hytale.server.core.modules.entity.component` |
| 15 | UUIDComponent | `com.hypixel.hytale.server.core.entity` |
| 17 | NetworkId | `com.hypixel.hytale.server.core.modules.entity.tracker` |
| 21 | Intangible | `com.hypixel.hytale.server.core.modules.entity.component` |
| 34 | SnapshotBuffer | `com.hypixel.hytale.server.core.modules.entity.component` |
| 41 | ModelComponent | `com.hypixel.hytale.server.core.modules.entity.component` |
| 42 | PersistentModel | `com.hypixel.hytale.server.core.modules.entity.component` |
| 50 | HiddenFromAdventurePlayers | `com.hypixel.hytale.server.core.modules.entity.component` |
| 53 | WorldGenId | `com.hypixel.hytale.server.core.modules.entity.component` |
| 60 | PrefabCopyableComponent | `com.hypixel.hytale.server.core.prefab` |
| 67 | PersistentRefCount | `com.hypixel.hytale.server.core.entity.reference` |
| 68 | Nameplate | `com.hypixel.hytale.server.core.entity.nameplate` |
| 131 | SpawnMarkerEntity | `com.hypixel.hytale.server.spawning.spawnmarkers` |

**Key components for entity identification:**
- `PrefabCopyableComponent` (60) - Likely contains the prefab/entity type ID
- `ModelComponent` (41) - Contains the 3D model reference
- `Nameplate` (68) - Display name for entities

### Official Docs Insights (hytale-docs.pages.dev)

**Component Access Pattern:**
```java
HealthComponent health = store.getComponent(entityRef, healthComponentType);
// Or ensure existence:
HealthComponent health = store.ensureAndGetComponent(entityRef, healthComponentType);
```

**Entity References (`Ref<ECS_TYPE>`):**
- Always check `isValid()` before using - references become invalid when entities removed
- Contains: store reference, index, validity status

**NPC/Creature System:**
- `NPCEntity` is base component for all NPCs
- Creatures are **asset-driven** - defined through asset files, not code
- `Role` class encapsulates complete behavior definitions
- Spawn components: `SpawnBeaconReference`, `SpawnMarkerReference`

**Implication:** Entity types (cow, pig, etc.) are determined by their asset configuration,
likely accessible through `PrefabCopyableComponent` or a model/asset reference.

### Entity Type Identification - SOLVED!

**The `ModelComponent.model` field contains `modelAssetId`** which identifies the entity type:

```java
// ModelComponent has a 'model' field with modelAssetId
// Examples found: Frog_Blue, Woodpecker, Duck, NPC_Path_Marker, Player

// Access pattern:
Object modelComp = chunk.getComponent(entityIndex, modelComponentType);
Field modelField = modelComp.getClass().getDeclaredField("model");
modelField.setAccessible(true);
Object model = modelField.get(modelComp);
// model.toString() shows: Model{modelAssetId='Duck', scale=1.0, ...}
```

**Known entity modelAssetIds (discovered in world):**

Farm Animals (breedable):
- `Cow` - Adult cow
- `Pig` - Adult pig
- `Piglet` - Baby pig
- `Boar_Piglet` - Wild baby boar
- `Boar` - Wild boar
- `Chicken` - Adult chicken
- `Chick` - Baby chicken
- `Sheep` - Adult sheep
- `Lamb` - Baby sheep

Wildlife:
- `Frog_Blue`, `Frog_Green` - Frogs
- `Woodpecker`, `Pigeon`, `Sparrow`, `Finch_Green`, `Owl_Brown`, `Tetrabird` - Birds
- `Duck` - Duck
- `Fox`, `Wolf_Black`, `Bear_Grizzly` - Predators
- `Bunny`, `Rabbit`, `Squirrel`, `Mouse` - Small animals
- `Model_Deer_Stag` - Deer
- `Spider` - Spider
- `Minnow`, `Bluegill`, `Trout_Rainbow` - Fish

System:
- `Player` - Player entity
- `NPC_Path_Marker`, `NPC_Spawn_Marker` - AI markers
- `Skeleton_Fighter` - Enemy NPC

### Iterating Entities on WorldThread

Entity iteration MUST run on the WorldThread. Use `World.execute(Runnable)`:

```java
Object player = ctx.sender();
Object world = callMethod(player, "getWorld");
Object entityStore = callMethod(world, "getEntityStore");
Object innerStore = callMethod(entityStore, "getStore");

// Find execute method
Method executeMethod = world.getClass().getMethod("execute", Runnable.class);

Runnable task = () -> {
    // Now on WorldThread - can use forEachChunk
    // Create BiConsumer proxy for iteration
    Object consumer = Proxy.newProxyInstance(...);
    forEachChunkMethod.invoke(store, consumer);
};

executeMethod.invoke(world, task);
```

---

## Interactions Component - For Right-Click Support

### Component Indices (on animals)
- **[72]**: `InteractionManager` - `com.hypixel.hytale.server.core.entity.InteractionManager`
- **[73]**: `Interactions` - `com.hypixel.hytale.server.core.modules.interaction.Interactions` ⭐
- **[74]**: `ChainingInteraction$Data` - chaining interaction data

### Interactions Class
Located at: `com.hypixel.hytale.server.core.modules.interaction.Interactions`

**Methods:**
- `getComponentType()` → ComponentType - Get the ComponentType for ECS access
- `getInteractions()` → Map - Get all interactions
- `setInteractionId(InteractionType, String)` → void - **KEY: Set interaction by type!**
- `getInteractionId(InteractionType)` → String - Get interaction ID for type
- `setInteractionHint(String)` → void
- `getInteractionHint()` → String
- `clone()` → Component
- `cloneSerializable()` → Component

**Fields:**
- `CODEC` : BuilderCodec
- `interactions` : Map - The interactions map
- `interactionHint` : String
- `isNetworkOutdated` : boolean

### How to Attach Custom Interaction (Tomorrow's Task)
```java
// 1. Get Interactions component from entity (index 73)
Object interactionsType = componentTypes[73];
Object interactionsComp = chunk.getComponent(entityIndex, interactionsType);

// 2. Call setInteractionId for right-click (Secondary = RMB)
// InteractionType.Secondary for right-click
interactionsComp.setInteractionId(InteractionType.Secondary, "MyCustomInteraction");

// 3. Your custom interaction (registered with ID "MyCustomInteraction") should then trigger
```

### InteractionType Values (from protocol)
- `Primary` - Left mouse button
- `Secondary` - Right mouse button ⭐
- `Use` - E key
- `Scroll` - Mouse scroll

---

## RootInteraction System - Deep Dive (v42)

### Key Findings

**RootInteraction AssetStore:**
- Located at: `com.hypixel.hytale.server.core.asset.HytaleAssetStore`
- Contains 5311+ RootInteractions
- Accessible via `RootInteraction.getAssetStore().getAssetMap()`

**ID Format in Store:**
- Keys have various prefixes: `*`, `**`, or none
- Example keys: `*Weapon_Longsword_...`, `**Outlander_...`, `Daggers_Stab_...`

**Critical Discovery:**
- `UseNPC` (used by animals as `*UseNPC`) is **NOT** in the RootInteraction store!
- `*UseNPC` refers to a **code-defined class** `UseNPCInteraction` at `com.hypixel.hytale.server.npc.interactions.UseNPCInteraction`
- The `*` prefix likely indicates a lookup in a different registry (Interaction classes, not RootInteraction assets)

**What Works:**
- Setting interaction ID on entities: `setInteractionId(InteractionType.Secondary, "MyCustomInteraction")` ✓
- Creating RootInteraction assets at `Item/RootInteractions/` ✓
- Creating Interaction assets at `Item/Interactions/` ✓
- Codec registration: `getCodecRegistry(Interaction.CODEC).register("MyCustomInteraction", ...)` ✓

**What Works Now:**
- Custom `SimpleInteraction` classes registered via `CodecMapRegistry`
- Asset files in `Server/Item/RootInteractions/` and `Server/Item/Interactions/`
- Entity interactions set via `setInteractionId()` at runtime (workaround)

**What Doesn't Work:**
- The `*` prefix mechanism for code-registered interactions (unclear how it works)
- `PrefabPlaceEntityEvent` / `LoadedNPCEvent` for detecting animal spawns

**Workarounds in Use:**
- Periodic entity scanning (30s) instead of spawn events
- Runtime interaction modification instead of NPC Role asset overrides

---

---

## Particle System (VERIFIED)

### Imports
```java
import com.hypixel.hytale.server.core.universe.world.ParticleUtil;
```

### Spawning Particles
```java
Vector3d position = new Vector3d(x, y, z);

// Use reflection to find correct method signature
for (Method method : ParticleUtil.class.getMethods()) {
    if (method.getName().equals("spawnParticleEffect") && method.getParameterCount() == 3) {
        Class<?>[] params = method.getParameterTypes();
        if (params[0] == String.class && params[1] == Vector3d.class) {
            method.invoke(null, "MyParticleSystem", position, store);
            break;
        }
    }
}
```

### Custom Particle Assets

**ParticleSystem:** `Server/Particles/MyParticle.particlesystem`
```json
{
  "Spawners": [
    {
      "SpawnerId": "MyParticle",
      "PositionOffset": { "X": 0, "Y": 0.4, "Z": -0.1 },
      "FixedRotation": false
    }
  ],
  "LifeSpan": 1
}
```

> **IMPORTANT:** Use `"Spawners"` array, NOT a direct `"SpawnerId"` field!

**ParticleSpawner:** `Server/Particles/Spawners/MyParticle.particlespawner`
```json
{
  "ParticleLifeSpan": { "Min": 0.8, "Max": 1 },
  "SpawnRate": { "Min": 5, "Max": 10 },
  "MaxConcurrentParticles": 5,
  "Particle": {
    "Texture": "Particles/Textures/Shapes/Hearts_HiRes.png",
    "FrameSize": { "Width": 64, "Height": 64 },
    "Animation": {
      "0": { "Color": "#fbbbfc", "Opacity": 1 },
      "100": { "Color": "#e296e3", "Opacity": 1 }
    }
  }
}
```

### Animation Keyframes

Animation uses percentage-based keyframes (0-100):
- `"0"` - Start of animation
- `"100"` - End of animation

Properties: `Color`, `Opacity`, `Scale`, `Rotation`, `FrameIndex`

---

## Sound System (VERIFIED)

### Imports
```java
import com.hypixel.hytale.server.core.asset.type.soundevent.config.SoundEvent;
import com.hypixel.hytale.server.core.universe.world.SoundUtil;
import com.hypixel.hytale.protocol.SoundCategory;
```

### Playing 3D Sounds
```java
int soundId = SoundEvent.getAssetMap().getIndex("SFX_Consume_Bread");
Vector3d pos = transform.getPosition();

// Direct call (works)
SoundUtil.playSoundEvent3d(soundId, SoundCategory.SFX,
    pos.getX(), pos.getY(), pos.getZ(), store);
```

---

## Entity Spawn Detection (VERIFIED)

Use `EntityTickingSystem` with `NewSpawnComponent` query for real-time spawn detection.

### Import
```java
import com.hypixel.hytale.component.system.tick.EntityTickingSystem;
```

### Implementation Pattern
```java
public class SpawnDetector extends EntityTickingSystem<EntityStore> {
    private static final ComponentType<EntityStore, ModelComponent> MODEL_TYPE =
        ModelComponent.getComponentType();

    // NewSpawnComponent requires reflection
    private static ComponentType<EntityStore, ?> newSpawnType = null;

    public SpawnDetector() {
        super();
        try {
            Class<?> cls = Class.forName(
                "com.hypixel.hytale.server.core.modules.entity.component.NewSpawnComponent");
            newSpawnType = (ComponentType<EntityStore, ?>) cls.getMethod("getComponentType").invoke(null);
        } catch (Exception e) { /* fallback to periodic scan */ }
    }

    @Override
    public void tick(float deltaTime, int entityIndex,
                     ArchetypeChunk<EntityStore> chunk,
                     Store<EntityStore> store,
                     CommandBuffer<EntityStore> buffer) {
        // Check if entity has NewSpawnComponent
        if (newSpawnType == null) return;
        Object comp = chunk.getComponent(entityIndex, newSpawnType);
        if (comp == null) return;

        // Get entity ref and model
        Ref<EntityStore> ref = chunk.getReferenceTo(entityIndex);
        ModelComponent model = chunk.getComponent(entityIndex, MODEL_TYPE);

        // Process newly spawned entity...
    }

    @Override
    public Query<EntityStore> getQuery() {
        // Query for entities with NewSpawnComponent
        return newSpawnType != null ? newSpawnType : MODEL_TYPE;
    }
}

// Register in setup():
getEntityStoreRegistry().registerSystem(new SpawnDetector());
```

> **Key insight:** `NewSpawnComponent` is added by the engine when entities spawn and removed after internal processing. Query for it to detect spawns in real-time.

---

## Movement System (VERIFIED)

### Overview

The player movement system consists of several interconnected components:

| Component | Class Path | Purpose |
|-----------|------------|---------|
| MovementManager | `com.hypixel.hytale.server.core.entity.entities.player.movement.MovementManager` | ECS component that manages player movement |
| MovementSettings | `com.hypixel.hytale.protocol.MovementSettings` | Protocol class containing movement parameters |
| MovementStatesComponent | `com.hypixel.hytale.server.core.entity.movement.MovementStatesComponent` | ECS component tracking movement state |
| MovementStates | `com.hypixel.hytale.protocol.MovementStates` | Protocol class with movement state flags |

### MovementManager (ECS Component)

Access via store:
```java
Class<?> mmClass = Class.forName("com.hypixel.hytale.server.core.entity.entities.player.movement.MovementManager");
Object compType = mmClass.getMethod("getComponentType").invoke(null);
Object movementManager = store.getComponent(playerRef, (ComponentType) compType);
```

**Methods:**
- `getSettings()` → MovementSettings - Get movement parameters
- `update(PacketHandler)` → void - Sync to client (expects `com.hypixel.hytale.server.core.io.PacketHandler`)

### MovementSettings Fields (Protocol)

All public fields on `com.hypixel.hytale.protocol.MovementSettings`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `canFly` | boolean | false | Enables flight (double-tap space) |
| `horizontalFlySpeed` | float | 10.32 | Horizontal fly speed |
| `verticalFlySpeed` | float | 10.32 | Vertical fly speed |
| `baseSpeed` | float | 5.5 | Base movement speed |
| `mass` | float | 68.0 | Player mass for physics |
| `jumpForce` | float | 11.8 | Jump strength |
| `invertedGravity` | boolean | false | Flip gravity |
| `collisionExpulsionForce` | float | 0.02 | Force when colliding |

**Speed Multipliers:**
- `forwardWalkSpeedMultiplier`, `backwardWalkSpeedMultiplier`, `strafeWalkSpeedMultiplier` (0.3)
- `forwardRunSpeedMultiplier` (1.0), `backwardRunSpeedMultiplier` (0.65), `strafeRunSpeedMultiplier` (0.8)
- `forwardCrouchSpeedMultiplier` (0.55), `forwardSprintSpeedMultiplier` (1.273)

**Example - Enable Flight:**
```java
Method getSettings = movementManager.getClass().getMethod("getSettings");
Object settings = getSettings.invoke(movementManager);
settings.getClass().getField("canFly").setBoolean(settings, true);
settings.getClass().getField("horizontalFlySpeed").setFloat(settings, 20.0f);
settings.getClass().getField("verticalFlySpeed").setFloat(settings, 20.0f);
```

### MovementStates Fields (Protocol)

State flags in `com.hypixel.hytale.protocol.MovementStates`:

| Field | Type | Description |
|-------|------|-------------|
| `idle` | boolean | Player is stationary |
| `horizontalIdle` | boolean | No horizontal movement |
| `jumping` | boolean | Currently jumping |
| `flying` | boolean | Currently in flight mode |
| `walking` | boolean | Walking speed |
| `running` | boolean | Running speed |
| `sprinting` | boolean | Sprint key held |
| `crouching` | boolean | Crouch key held |
| `forcedCrouching` | boolean | Forced crouch (low ceiling) |
| `falling` | boolean | In freefall |
| `climbing` | boolean | On ladder/climbable |
| `inFluid` | boolean | In water/liquid |
| `swimming` | boolean | Swimming in liquid |
| `swimJumping` | boolean | Jump while swimming |
| `onGround` | boolean | Touching ground |
| `mantling` | boolean | Climbing over ledge |
| `sliding` | boolean | Slide action |
| `mounting` | boolean | On mount |
| `rolling` | boolean | Roll action |
| `sitting` | boolean | Sitting |
| `gliding` | boolean | Glider active |
| `sleeping` | boolean | In bed |

### Intangible / Invulnerable Components

Flag components that can be added/removed:

```java
// Intangible - makes entity pass through other entities (NOT blocks!)
Class<?> intangibleClass = Class.forName("com.hypixel.hytale.server.core.modules.entity.component.Intangible");
Object compType = intangibleClass.getMethod("getComponentType").invoke(null);
store.ensureAndGetComponent(playerRef, (ComponentType) compType); // Add
store.removeComponent(playerRef, (ComponentType) compType);       // Remove

// Invulnerable - prevents damage
Class<?> invulnClass = Class.forName("com.hypixel.hytale.server.core.modules.entity.component.Invulnerable");
// Same pattern as above
```

> **Note:** `Intangible` affects entity-entity collision only, NOT block collision!

### GameMode System

```java
Class<?> gameModeClass = Class.forName("com.hypixel.hytale.protocol.GameMode");
// Enum values: Creative, Adventure, Survival (check at runtime)

// Set on player
for (Field f : player.getClass().getDeclaredFields()) {
    if (f.getType() == gameModeClass) {
        f.setAccessible(true);
        f.set(player, targetMode);
        break;
    }
}

// Send packet to client
Class<?> packetClass = Class.forName("com.hypixel.hytale.protocol.packets.player.SetGameMode");
Object packet = packetClass.getConstructors()[0].newInstance(targetMode); // 1-param constructor
Object connection = player.getClass().getMethod("getPlayerConnection").invoke(player);
connection.getClass().getMethod("write", Object.class).invoke(connection, packet);
```

### No-Clip Collision (CLIENT-SIDE)

**Critical Finding:** Block collision disable (noclip) appears to be **client-authoritative**.

- The creative mode UI has a noclip toggle button
- No server-side field or packet was found to control this
- `MovementSettings` has no noclip/collision field
- `MovementStates` has no noclip/collision field
- `Intangible` component only affects entity-entity collision

**Implication:** True noclip (passing through blocks) may require:
1. Client-side mod
2. An undiscovered server packet
3. The creative UI toggle being client-only

---

---

## NPC Behavior System (VERIFIED)

### Core Components Registration

Register custom actions/sensors for NPC behavior trees in `start()`:

```java
NPCPlugin.get().registerCoreComponentType("Tame", BuilderActionTame::new);
NPCPlugin.get().registerCoreComponentType("Tamed", BuilderSensorTamed::new);
```

### Class Paths (NPC System)

| Class | Path |
|-------|------|
| `NPCPlugin` | `com.hypixel.hytale.server.npc.NPCPlugin` |
| `NPCEntity` | `com.hypixel.hytale.server.npc.entities.NPCEntity` |
| `Role` | `com.hypixel.hytale.server.npc.role.Role` |
| `WorldSupport` | `com.hypixel.hytale.server.npc.role.support.WorldSupport` |
| `StateSupport` | `com.hypixel.hytale.server.npc.role.support.StateSupport` |
| `Attitude` | `com.hypixel.hytale.server.core.asset.type.attitude.Attitude` |
| `AttitudeGroup` | `com.hypixel.hytale.server.npc.config.AttitudeGroup` |
| `ActionBase` | `com.hypixel.hytale.server.npc.corecomponents.ActionBase` |
| `SensorBase` | `com.hypixel.hytale.server.npc.corecomponents.SensorBase` |
| `BuilderActionBase` | `com.hypixel.hytale.server.npc.corecomponents.builders.BuilderActionBase` |
| `BuilderSensorBase` | `com.hypixel.hytale.server.npc.corecomponents.builders.BuilderSensorBase` |
| `BuilderSupport` | `com.hypixel.hytale.server.npc.asset.builder.BuilderSupport` |
| `InfoProvider` | `com.hypixel.hytale.server.npc.sensorinfo.InfoProvider` |
| `RoleBuilderSystem` | `com.hypixel.hytale.server.npc.systems.RoleBuilderSystem` |

### Class Paths (ECS Systems)

| Class | Path |
|-------|------|
| `HolderSystem` | `com.hypixel.hytale.component.system.HolderSystem` |
| `Holder` | `com.hypixel.hytale.component.Holder` |
| `Query` | `com.hypixel.hytale.component.query.Query` |
| `Dependency` | `com.hypixel.hytale.component.dependency.Dependency` |
| `SystemDependency` | `com.hypixel.hytale.component.dependency.SystemDependency` |
| `Order` | `com.hypixel.hytale.component.dependency.Order` |
| `AddReason` | `com.hypixel.hytale.component.AddReason` |
| `RemoveReason` | `com.hypixel.hytale.component.RemoveReason` |

### Class Paths (Codecs)

| Class | Path |
|-------|------|
| `BuilderCodec` | `com.hypixel.hytale.codec.builder.BuilderCodec` |
| `Codec` | `com.hypixel.hytale.codec.Codec` |
| `KeyedCodec` | `com.hypixel.hytale.codec.KeyedCodec` |

### Holder Types (for config parsing)

| Class | Path |
|-------|------|
| `StringArrayHolder` | `com.hypixel.hytale.server.npc.asset.builder.holder.StringArrayHolder` |
| `BooleanHolder` | `com.hypixel.hytale.server.npc.asset.builder.holder.BooleanHolder` |
| `BuilderDescriptorState` | `com.hypixel.hytale.server.npc.asset.builder.BuilderDescriptorState` |

### NPC Spawn Tracking

Remove tamed animals from spawn population limits:

```java
NPCEntity npc = store.getComponent(ref, NPCEntity.getComponentType());
boolean wasTracked = npc.updateSpawnTrackingState(false);  // Remove from tracking
npc.updateSpawnTrackingState(true);  // Re-enable tracking
```

### Attitude System

Set NPC attitude to friendly (requires reflection):

```java
// Cache field
Field attitudeField = WorldSupport.class.getDeclaredField("defaultPlayerAttitude");
attitudeField.setAccessible(true);

// Get WorldSupport from NPC
NPCEntity npc = store.getComponent(ref, NPCEntity.getComponentType());
WorldSupport worldSupport = npc.getRole().getWorldSupport();

// Set attitude
attitudeField.set(worldSupport, Attitude.REVERED);
```

### Attitude Groups (discovered)

| Group ID | Animals |
|----------|---------|
| `Livestock` | Cow, Pig, Chicken, Sheep |
| `PreyBig` | Deer, Horse, Camel |
| `PreySmall` | Rabbit, Bunny |
| `Critters` | Frog, Mouse, Squirrel |
| `Predator` | Wolf, Bear, Fox |

---

## TODO: Continue discovering
- [x] How to access world from player - Use `ctx.sender().getWorld()`
- [x] How to iterate all entities in a world - Use `World.execute()` + `Store.forEachChunk()`
- [x] How to identify entity types - Use `ModelComponent.model.modelAssetId`
- [x] Interactions component location - Index 73, has `setInteractionId()`
- [x] Attach custom interaction to animals at runtime - Works via `setInteractionId()` + asset files
- [x] Particle system spawning - Use `ParticleUtil.spawnParticleEffect()` + asset files
- [x] Sound system - Use `SoundUtil.playSoundEvent3d()` with `SoundCategory`
- [x] Entity spawn detection - Use `EntityTickingSystem` + `NewSpawnComponent` query
- [x] Movement system - MovementManager, MovementSettings, MovementStates components
- [x] Flight control - Set `canFly` in MovementSettings
- [x] NPC Behavior Trees - Custom actions/sensors via `NPCPlugin.registerCoreComponentType()`
- [x] NPC Attitude System - Set via reflection on `WorldSupport.defaultPlayerAttitude`
- [x] NPC Spawn Tracking - `NPCEntity.updateSpawnTrackingState(false)` to remove from limits
- [x] Custom ECS Components - Use `BuilderCodec` for persistent components
- [x] HolderSystem - Entity lifecycle system (onEntityAdd/onEntityRemoved)
- [ ] Figure out `*` prefix mechanism for code-registered interactions
- [ ] NPC Role asset overrides for interactions (instead of runtime modification)
- [ ] Block collision disable (noclip) - appears to be client-side only
