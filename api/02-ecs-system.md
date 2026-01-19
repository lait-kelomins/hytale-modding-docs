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

## Related Files

- [01-plugin-system.md](api/01-plugin-system.md) - Plugin setup
- [03-entities.md](api/03-entities.md) - Entity details
- [05-events.md](api/05-events.md) - Player events
