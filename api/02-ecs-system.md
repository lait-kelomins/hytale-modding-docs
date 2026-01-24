# Hytale ECS (Entity Component System)

> **Index:** [Overview](#overview) | [EntityEventSystem](#entityeventsystem) | [Components](#components) | [Query](#query-system) | [Events](#available-events) | [Example](#complete-example)

---

## Overview

Hytale uses an ECS architecture. Instead of traditional event listeners, you create **Systems** that process entities with specific components when events occur.

**Key Classes:**
```
com.hypixel.hytale.component.system.EntityEventSystem  # Handle entity events
com.hypixel.hytale.component.system.WorldEventSystem   # Handle world events
com.hypixel.hytale.component.system.TickingSystem      # Periodic updates
com.hypixel.hytale.component.ArchetypeChunk            # Entity data container
com.hypixel.hytale.component.CommandBuffer             # Deferred commands
com.hypixel.hytale.component.Store                     # World/entity store
com.hypixel.hytale.component.query.Query               # Component queries
com.hypixel.hytale.component.ComponentType             # Component type reference
```

---

## EntityEventSystem

The primary way to handle game events. Extend this class to react to specific events.

### Signature

```java
public abstract class EntityEventSystem<S extends Store, E extends EcsEvent>
```

- `S` - Store type (usually `EntityStore`)
- `E` - Event type (e.g., `BreakBlockEvent`, `UseBlockEvent`)

### Methods to Implement

```java
// Constructor - pass the event class
protected MySystem() {
    super(BreakBlockEvent.class);
}

// Handle the event for each matching entity
@Override
public void handle(
    int entityIndex,                        // Index in chunk
    ArchetypeChunk<EntityStore> chunk,      // Entity data
    Store<EntityStore> store,               // World store
    CommandBuffer<EntityStore> buffer,      // Deferred commands
    BreakBlockEvent event                   // The event
) {
    // Your logic here
}

// Define which entities this system processes
@Override
public Query<EntityStore> getQuery() {
    return PlayerRef.getComponentType();  // Only entities with PlayerRef
}
```

---

## EntityTickingSystem (VERIFIED)

For periodic processing of entities (e.g., detecting spawns, updating state). Runs every tick on matching entities.

### Import
```java
import com.hypixel.hytale.component.system.tick.EntityTickingSystem;
```

### Signature
```java
public abstract class EntityTickingSystem<S extends Store>
```

### Methods to Implement
```java
public class MyTickingSystem extends EntityTickingSystem<EntityStore> {

    public MyTickingSystem() {
        super();
    }

    @Override
    public void tick(
        float deltaTime,                        // Time since last tick
        int entityIndex,                        // Index in chunk
        ArchetypeChunk<EntityStore> chunk,      // Entity data
        Store<EntityStore> store,               // World store
        CommandBuffer<EntityStore> buffer       // Deferred commands
    ) {
        // Your per-entity logic here
    }

    @Override
    public Query<EntityStore> getQuery() {
        return MyComponent.getComponentType();  // Only entities with MyComponent
    }
}
```

### Use Case: Spawn Detection
Query for `NewSpawnComponent` to detect newly spawned entities:
```java
// NewSpawnComponent is added by engine on entity creation
Class<?> cls = Class.forName(
    "com.hypixel.hytale.server.core.modules.entity.component.NewSpawnComponent");
ComponentType<EntityStore, ?> spawnType =
    (ComponentType<EntityStore, ?>) cls.getMethod("getComponentType").invoke(null);

// Return as query to only tick on newly spawned entities
return spawnType;
```

### Registration
```java
@Override
protected void setup() {
    getEntityStoreRegistry().registerSystem(new MyTickingSystem());
}
```

---

## Components

Entities are collections of components. Access components from the ArchetypeChunk.

### Getting Components

```java
@Override
public void handle(int i, ArchetypeChunk<EntityStore> chunk, ...) {
    // Get a component for entity at index i
    PlayerRef playerRef = chunk.getComponent(i, PlayerRef.getComponentType());

    // Use the component
    String username = playerRef.getUsername();
}
```

### Common Component Types

```
com.hypixel.hytale.server.core.universe.PlayerRef     # Player reference
```

### ComponentType Pattern

Components provide a static `getComponentType()` method:

```java
ComponentType<PlayerRef> type = PlayerRef.getComponentType();
```

### Direct vs Reflection Access

> **VERIFIED:** Some components work with direct `getComponentType()` calls, others require reflection.

**Direct Access (works):**
```java
import com.hypixel.hytale.server.core.modules.entity.component.TransformComponent;
import com.hypixel.hytale.server.core.modules.entity.component.ModelComponent;
import com.hypixel.hytale.server.core.entity.UUIDComponent;

// Cache as static fields for performance
private static final ComponentType<EntityStore, TransformComponent> TRANSFORM_TYPE =
    TransformComponent.getComponentType();
private static final ComponentType<EntityStore, ModelComponent> MODEL_TYPE =
    ModelComponent.getComponentType();
private static final ComponentType<EntityStore, UUIDComponent> UUID_TYPE =
    UUIDComponent.getComponentType();

// Usage
TransformComponent transform = store.getComponent(entityRef, TRANSFORM_TYPE);
```

**Require Reflection:**
```java
import com.hypixel.hytale.server.core.modules.entity.component.Interactable;
import com.hypixel.hytale.server.core.modules.interaction.Interactions;

// These require reflection for getComponentType()
Object interactableType = Interactable.class.getMethod("getComponentType").invoke(null);
Object interactionsType = Interactions.class.getMethod("getComponentType").invoke(null);

// Then use with store
Object interactions = store.ensureAndGetComponent(entityRef, interactionsType);
```

---

## Query System

Queries filter which entities a system processes.

### Basic Queries

```java
// Process entities with PlayerRef component
@Override
public Query<EntityStore> getQuery() {
    return PlayerRef.getComponentType();
}
```

### Query Combinators

```java
import com.hypixel.hytale.component.query.*;

// AND - requires both components
new AndQuery<>(ComponentA.getComponentType(), ComponentB.getComponentType())

// OR - requires either component
new OrQuery<>(ComponentA.getComponentType(), ComponentB.getComponentType())

// NOT - excludes component
new NotQuery<>(ComponentA.getComponentType())

// ANY - matches all entities
new AnyQuery<>()
```

---

## Available Events

### Block Events (`com.hypixel.hytale.server.core.event.events.ecs`)

| Event | Trigger |
|-------|---------|
| `BreakBlockEvent` | Player breaks a block |
| `PlaceBlockEvent` | Player places a block |
| `DamageBlockEvent` | Block takes damage |
| `UseBlockEvent` | Player interacts with block |
| `UseBlockEvent.Pre` | Before interaction |
| `UseBlockEvent.Post` | After interaction |

### Craft Events

| Event | Trigger |
|-------|---------|
| `CraftRecipeEvent` | Player crafts item |
| `CraftRecipeEvent.Pre` | Before crafting |
| `CraftRecipeEvent.Post` | After crafting |

### Item Events

| Event | Trigger |
|-------|---------|
| `DropItemEvent` | Item dropped |
| `InteractivelyPickupItemEvent` | Item picked up |
| `SwitchActiveSlotEvent` | Hotbar slot changed |

---

## CommandBuffer

Use CommandBuffer for deferred entity operations (creating, modifying, removing entities).

```java
@Override
public void handle(..., CommandBuffer<EntityStore> buffer, ...) {
    // Queue operations to execute after event processing
    // buffer.createEntity(...)
    // buffer.addComponent(...)
    // buffer.removeEntity(...)
}
```

---

## Complete Example

From the original docs - a system that handles block breaking:

```java
package com.example.myplugin;

import com.hypixel.hytale.component.ArchetypeChunk;
import com.hypixel.hytale.component.CommandBuffer;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.component.query.Query;
import com.hypixel.hytale.component.system.EntityEventSystem;
import com.hypixel.hytale.server.core.event.events.ecs.BreakBlockEvent;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

public class BlockBreakHandler extends EntityEventSystem<EntityStore, BreakBlockEvent> {

    public BlockBreakHandler() {
        super(BreakBlockEvent.class);
    }

    @Override
    public void handle(
            int i,
            @NotNull ArchetypeChunk<EntityStore> chunk,
            @NotNull Store<EntityStore> store,
            @NotNull CommandBuffer<EntityStore> buffer,
            @NotNull BreakBlockEvent event
    ) {
        // Get the player who broke the block
        PlayerRef playerRef = chunk.getComponent(i, PlayerRef.getComponentType());

        System.out.println("Block broken by: " + playerRef.getUsername());

        // Access event data
        // event.getBlockPosition(), event.getBlockType(), etc.
    }

    @Nullable
    @Override
    public Query<EntityStore> getQuery() {
        // Only process for entities with PlayerRef (players)
        return PlayerRef.getComponentType();
    }
}
```

---

## Registering Systems

> **WARNING:** The correct method is `setup()`, NOT `onEnable()` (Bukkit pattern). Hytale uses different lifecycle methods.

In your plugin's `setup()` method:

```java
@Override
protected void setup() {
    // Register ECS systems via EntityStoreRegistry
    getEntityStoreRegistry().registerSystem(new MyEntityEventSystem());
}
```

> **NOTE:** Custom ECS system registration is **untested**. The API exists but practical examples are limited.

---

## HolderSystem (Entity Lifecycle)

For processing entities when they are added to or removed from the world. Unlike `EntityEventSystem` which handles events, `HolderSystem` handles entity lifecycle.

### Import
```java
import com.hypixel.hytale.component.system.HolderSystem;
import com.hypixel.hytale.component.Holder;
import com.hypixel.hytale.component.AddReason;
import com.hypixel.hytale.component.RemoveReason;
import com.hypixel.hytale.component.dependency.*;
```

### Signature
```java
public abstract class HolderSystem<S extends Store>
```

### Implementation
```java
public class TameActivateSystem extends HolderSystem<EntityStore> {
    private final ComponentType<EntityStore, NPCEntity> npcType;
    private final ComponentType<EntityStore, TameComponent> tameType;
    private final Query<EntityStore> query;
    private final Set<Dependency<EntityStore>> dependencies;

    public TameActivateSystem() {
        this.npcType = NPCEntity.getComponentType();
        this.tameType = TameComponent.getComponentType();

        // Query: entities with NPCEntity but NOT NPCMountComponent
        this.query = Query.and(npcType, Query.not(NPCMountComponent.getComponentType()));

        // Run AFTER RoleBuilderSystem completes
        this.dependencies = Set.of(new SystemDependency<>(Order.AFTER, RoleBuilderSystem.class));
    }

    @Override
    public Query<EntityStore> getQuery() { return this.query; }

    @Override
    public Set<Dependency<EntityStore>> getDependencies() { return this.dependencies; }

    @Override
    public void onEntityAdd(Holder<EntityStore> holder, AddReason reason, Store<EntityStore> store) {
        // Entity was added to world (spawned or loaded)
        NPCEntity npc = holder.getComponent(npcType);
        TameComponent tame = holder.ensureAndGetComponent(tameType);

        if (tame.isTamed()) {
            // Restore tamed state after world load
        }
    }

    @Override
    public void onEntityRemoved(Holder<EntityStore> holder, RemoveReason reason, Store<EntityStore> store) {
        // Entity removed from world - cleanup if needed
    }
}
```

### Key Differences from EntityEventSystem
| Feature | EntityEventSystem | HolderSystem |
|---------|-------------------|--------------|
| Trigger | Specific events (BreakBlockEvent, etc.) | Entity add/remove |
| Use Case | React to player actions | Entity lifecycle, state restoration |
| Holder Access | Via ArchetypeChunk | Direct Holder parameter |

### Registration
```java
@Override
protected void setup() {
    getEntityStoreRegistry().registerSystem(new TameActivateSystem());
}
```

---

## Custom Components (Persistence)

Create components that save/load with the world using `BuilderCodec`.

### Import
```java
import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import com.hypixel.hytale.component.Component;
import com.hypixel.hytale.component.ComponentType;
```

### Component Definition
```java
public class TameComponent implements Component<EntityStore> {
    // CODEC defines serialization (saved to world)
    public static final BuilderCodec<TameComponent> CODEC = BuilderCodec.builder(
            TameComponent.class, TameComponent::new)
        .append(new KeyedCodec<>("IsTamed", Codec.BOOLEAN),
            (data, val) -> data.isTamed = val, data -> data.isTamed)
        .add()
        .append(new KeyedCodec<>("TamerUUID", Codec.UUID_BINARY),
            (data, val) -> data.tamerUUID = val, data -> data.tamerUUID)
        .add()
        .append(new KeyedCodec<>("TamerName", Codec.STRING),
            (data, val) -> data.tamerName = val, data -> data.tamerName)
        .add()
        .build();

    // Use boxed Boolean for null safety in codec
    private Boolean isTamed = false;
    private UUID tamerUUID = null;
    private String tamerName = null;

    public boolean isTamed() {
        return Boolean.TRUE.equals(isTamed);
    }

    public void setTamed(UUID player, String playerName) {
        this.isTamed = true;
        this.tamerUUID = player;
        this.tamerName = playerName;
    }

    @Override
    public Component<EntityStore> clone() {
        TameComponent copy = new TameComponent();
        copy.isTamed = this.isTamed;
        copy.tamerUUID = this.tamerUUID;
        copy.tamerName = this.tamerName;
        return copy;
    }
}
```

### Registration (in Plugin)
```java
public class MyPlugin extends JavaPlugin {
    private ComponentType<EntityStore, TameComponent> tameComponentType;

    @Override
    protected void setup() {
        // Register component with name and codec
        tameComponentType = this.getEntityStoreRegistry().registerComponent(
            TameComponent.class, "Tame", TameComponent.CODEC);
    }

    public ComponentType<EntityStore, TameComponent> getTameComponentType() {
        return tameComponentType;
    }
}
```

### Usage
```java
// Get component (returns null if missing)
TameComponent comp = store.getComponent(entityRef, tameComponentType);

// Get or create component
TameComponent comp = store.ensureAndGetComponent(entityRef, tameComponentType);

// From Holder (in HolderSystem)
TameComponent comp = holder.getComponent(tameComponentType);
TameComponent comp = holder.ensureAndGetComponent(tameComponentType);
```

### Available Codec Types
| Codec | Java Type |
|-------|-----------|
| `Codec.BOOLEAN` | Boolean |
| `Codec.STRING` | String |
| `Codec.INT` | Integer |
| `Codec.LONG` | Long |
| `Codec.FLOAT` | Float |
| `Codec.DOUBLE` | Double |
| `Codec.UUID_BINARY` | UUID |

---

## Related Files

- [01-plugin-system.md](api/01-plugin-system.md) - Plugin setup
- [03-entities.md](api/03-entities.md) - Entity details
- [05-events.md](api/05-events.md) - Player events
