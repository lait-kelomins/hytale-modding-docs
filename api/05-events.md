# Hytale Event System

> **Source:** https://hytale-docs.com/docs/api/server-internals/events

---

## Overview

The event system uses an EventBus pattern with support for synchronous, asynchronous, and ECS events.

**Event Hierarchy:**
```
IBaseEvent<KeyType>
├── IEvent<KeyType>          (synchronous)
└── IAsyncEvent<KeyType>     (asynchronous)
```

The generic `KeyType` allows filtering events by key (e.g., player UUID).

**IMPORTANT:** All events must be registered in `setup()`, NOT in `start()`!

---

## Registration Methods

### Standard Registration
```java
// Basic registration
getEventRegistry().register(EventClass.class, event -> { /* handler */ });

// With specific key (only events matching this key)
getEventRegistry().register(EventClass.class, keyValue, event -> { /* handler */ });
```

### Priority-Based Registration
```java
// Using EventPriority enum
getEventRegistry().register(EventPriority.EARLY, EventClass.class, handler);

// Using raw priority value
getEventRegistry().register((short) -100, EventClass.class, handler);
```

**Priority Values:**

| Priority | Value | Use Case |
|----------|-------|----------|
| `FIRST` | -21844 | Security checks, validation |
| `EARLY` | -10922 | Event transformation |
| `NORMAL` | 0 | Standard logic (default) |
| `LATE` | 10922 | Logging, monitoring |
| `LAST` | 21844 | Cleanup, final processing |

### Global Registration
Invoked for ALL events of a type, regardless of key:
```java
getEventRegistry().registerGlobal(PlayerChatEvent.class, event -> { });
```

### Unhandled Registration
Invoked ONLY if no other handler processed the event:
```java
getEventRegistry().registerUnhandled(PlayerInteractEvent.class, event -> { });
```

### Async Registration
For non-blocking operations:
```java
getEventRegistry().registerAsync(AsyncEvent.class, future -> {
    return future.thenApply(event -> {
        // async processing
        return event;
    });
});
```

---

## Event Types

### Synchronous Events (IEvent)
- Blocking, sequential execution
- Handler signature: `void handler(EventType event)`

### Asynchronous Events (IAsyncEvent)
- Non-blocking with `CompletableFuture`
- Handler signature: `Function<CompletableFuture<E>, CompletableFuture<E>>`

### Cancellable Events (ICancellable)
```java
public interface ICancellable {
    boolean isCancelled();
    void setCancelled(boolean cancelled);
}
```

**Cancellable events include:** `PlayerChatEvent`, `PlayerInteractEvent`, `PlayerSetupConnectEvent`, `PlayerMouseButtonEvent`, `BreakBlockEvent`, `PlaceBlockEvent`, `DropItemEvent`, `CraftRecipeEvent`

---

## Core Server Events

**Package:** `com.hypixel.hytale.server.core.event.events`

| Event | Type | Cancellable | Purpose |
|-------|------|-------------|---------|
| `BootEvent` | Sync | No | Server started |
| `ShutdownEvent` | Sync | No | Server shutting down |
| `PrepareUniverseEvent` | Sync | No | Universe preparation |

---

## Player Events

**Package:** `com.hypixel.hytale.server.core.event.events.player`

| Event | Type | Cancellable | Purpose |
|-------|------|-------------|---------|
| `PlayerConnectEvent` | Sync | No | Connection established |
| `PlayerSetupConnectEvent` | Sync | Yes | Pre-connection (auth) |
| `PlayerDisconnectEvent` | Sync | No | Disconnection |
| `PlayerSetupDisconnectEvent` | Sync | No | Disconnect cleanup |
| `PlayerReadyEvent` | Sync | No | Player ready in world |
| `PlayerChatEvent` | Async | Yes | Chat message |
| `PlayerInteractEvent` | Sync | Yes | Entity interaction |
| `PlayerCraftEvent` | Sync | No | Item crafted |
| `PlayerMouseButtonEvent` | Sync | Yes | Mouse button input |
| `PlayerMouseMotionEvent` | Sync | No | Mouse movement |
| `AddPlayerToWorldEvent` | Sync | No | Player added to world |
| `DrainPlayerFromWorldEvent` | Sync | No | Player removed from world |

---

## Block/Item Events (ECS)

**Package:** `com.hypixel.hytale.server.core.event.events.ecs`

| Event | Type | Cancellable | Purpose |
|-------|------|-------------|---------|
| `UseBlockEvent.Pre` | ECS | Yes | Before block interaction |
| `UseBlockEvent.Post` | ECS | No | After block interaction |
| `BreakBlockEvent` | ECS | Yes | Block destruction |
| `PlaceBlockEvent` | ECS | Yes | Block placement |
| `DamageBlockEvent` | ECS | Yes | Block damage |
| `DropItemEvent.PlayerRequest` | ECS | Yes | Drop request |
| `DropItemEvent.Drop` | ECS | No | Drop action |
| `InteractivelyPickupItemEvent` | ECS | No | Item picked up |
| `SwitchActiveSlotEvent` | ECS | No | Hotbar slot change |
| `CraftRecipeEvent.Pre` | ECS | Yes | Pre-craft |
| `CraftRecipeEvent.Post` | ECS | No | Post-craft |
| `ChangeGameModeEvent` | ECS | No | Game mode change |
| `DiscoverZoneEvent` | ECS | No | Zone discovered |

### Block Event Registration
```java
// Use register(), NOT registerGlobal() for block events
getEventRegistry().register(UseBlockEvent.Pre.class, this::onUseBlockPre);
getEventRegistry().register(UseBlockEvent.Post.class, this::onUseBlockPost);
```

---

## Entity Events

**Package:** `com.hypixel.hytale.server.core.event.events.entity`

| Event | Description |
|-------|-------------|
| `EntityEvent` | Base entity event |
| `EntityRemoveEvent` | Entity removed from world |
| `LivingEntityInventoryChangeEvent` | Inventory changes |
| `LivingEntityUseBlockEvent` | Entity uses block |

---

## World/Universe Events

**Package:** `com.hypixel.hytale.server.core.universe.world.events.ecs`

| Event | Description |
|-------|-------------|
| `ChunkSaveEvent` | Chunk being saved |
| `ChunkUnloadEvent` | Chunk being unloaded |
| `MoonPhaseChangeEvent` | Moon phase changes |

---

## Permission Events

**Package:** `com.hypixel.hytale.server.core.event.events.permissions`

| Event | Description |
|-------|-------------|
| `PlayerPermissionChangeEvent` | Player permissions change |
| `PlayerGroupEvent` | Player added/removed from group |
| `GroupPermissionChangeEvent` | Group permissions change |

---

## Plugin Events

**Package:** `com.hypixel.hytale.server.core.plugin.event`

| Event | Description |
|-------|-------------|
| `PluginEvent` | Base plugin event |
| `PluginSetupEvent` | Plugin setup phase |

---

## Unregistration

```java
// Get registration handle
EventRegistration<Void, BootEvent> reg = getEventRegistry().register(...);

// Unregister when done
reg.unregister();

// Check if still active
if (reg.isEnabled()) { /* still registered */ }

// Combine multiple registrations
EventRegistration<?, ?> combined = EventRegistration.combine(reg1, reg2, reg3);
combined.unregister(); // Unregisters all
```

---

## Dispatching Custom Events

### Synchronous Dispatch
```java
EventType result = eventBus.dispatchFor(EventClass.class).dispatch(event);
EventType result = eventBus.dispatchFor(EventClass.class, key).dispatch(event);
```

### Async Dispatch
```java
CompletableFuture<EventType> future =
    eventBus.dispatchForAsync(EventClass.class).dispatch(event);

future.whenComplete((event, error) -> { });
```

### Check for Listeners
```java
if (eventBus.dispatchFor(EventClass.class).hasListener()) {
    // Has registered handlers
}
```

---

## Creating Custom Events

### Synchronous with Key
```java
public class PlayerScoreEvent implements IEvent<UUID> {
    private final UUID playerId;
    private final int score;

    public PlayerScoreEvent(UUID playerId, int score) {
        this.playerId = playerId;
        this.score = score;
    }

    public UUID getPlayerId() { return playerId; }
    public int getScore() { return score; }
}
```

### Cancellable Event
```java
public class PlayerTeleportEvent implements IEvent<Void>, ICancellable {
    private boolean cancelled = false;
    private final Player player;
    private final Location destination;

    @Override
    public boolean isCancelled() { return cancelled; }

    @Override
    public void setCancelled(boolean cancelled) {
        this.cancelled = cancelled;
    }
}
```

---

## Execution Order

1. Handlers sorted by priority (ascending short value)
2. Within same priority, registration order preserved
3. Global handlers execute after key-specific handlers
4. Unhandled handlers only if no prior handler processed

---

## Best Practices

| Priority | Use For |
|----------|---------|
| FIRST | Security checks, permission validation |
| EARLY | Event transformation, modification |
| NORMAL | Core business logic |
| LATE | Logging, analytics, monitoring |
| LAST | Cleanup, final state changes |

**Guidelines:**
- Always unregister handlers in `shutdown()`
- Check `isCancelled()` before processing cancellable events
- Use async handlers for database/network operations
- Never rethrow exceptions - allow other handlers to execute

---

## Example: Complete Event Handler

```java
@Override
protected void setup() {
    // Block interaction events (use register, not registerGlobal)
    getEventRegistry().register(UseBlockEvent.Pre.class, this::onUseBlockPre);
    getEventRegistry().register(UseBlockEvent.Post.class, this::onUseBlockPost);

    // Player events (global - all players)
    getEventRegistry().registerGlobal(PlayerReadyEvent.class, this::onPlayerReady);

    // Chat with priority
    getEventRegistry().register(
        EventPriority.EARLY,
        PlayerChatEvent.class,
        this::onChat
    );
}

private void onUseBlockPre(UseBlockEvent.Pre event) {
    getLogger().atInfo().log("Block used: " + event);
    // event.setCancelled(true); // Can cancel
}

private void onUseBlockPost(UseBlockEvent.Post event) {
    getLogger().atInfo().log("Block use complete");
}

private void onPlayerReady(PlayerReadyEvent event) {
    Player player = event.getPlayer();
    player.sendMessage(Message.raw("Welcome!"));
}

private void onChat(PlayerChatEvent event) {
    if (event.isCancelled()) return;
    // Process chat
}
```

---

## Related Files

- [01-plugin-system.md](./01-plugin-system.md) - Plugin lifecycle
- [02-ecs-system.md](./02-ecs-system.md) - ECS events
- [06-interactions.md](./06-interactions.md) - Interaction system
