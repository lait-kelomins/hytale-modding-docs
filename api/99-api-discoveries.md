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

## TODO: Continue discovering
- [x] How to access world from player - Use `ctx.sender().getWorld()`
- [x] How to iterate all entities in a world - Use `World.execute()` + `Store.forEachChunk()`
- [x] How to identify entity types - Use `ModelComponent.model.modelAssetId`
- [x] Interactions component location - Index 73, has `setInteractionId()`
- [x] Attach custom interaction to animals at runtime - Works via `setInteractionId()` + asset files
- [x] Particle system spawning - Use `ParticleUtil.spawnParticleEffect()` + asset files
- [x] Sound system - Use `SoundUtil.playSoundEvent3d()` with `SoundCategory`
- [ ] Figure out `*` prefix mechanism for code-registered interactions
- [ ] Find proper entity spawn detection (NewSpawnComponent + TickingSystem?)
- [ ] NPC Role asset overrides for interactions (instead of runtime modification)
