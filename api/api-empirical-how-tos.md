# Hytale Server API Discoveries (Runtime Exploration)

These are actual API findings from runtime reflection and testing on HytaleServer.jar. Many of these differ from or extend official documentation.

> **Note:** This document was created during mod development and reflects discoveries made through experimentation. Some paths or behaviors may change in future Hytale versions.

---

## Table of Contents
- [How-To Tutorials](#how-to-tutorials)
- [CommandContext](#commandcontext)
- [HytaleServer](#hytaleserver)
- [Interaction System](#interaction-system)
- [ItemStack](#itemstack)
- [World/Universe](#worlduniverse)
- [Entity References](#entity-references)
- [Events](#events)
- [ECS Component System](#ecs-component-system)
- [Interactions Component](#interactions-component---for-right-click-support)
- [RootInteraction System](#rootinteraction-system)

---

# How-To Tutorials

Practical, copy-paste-ready examples for common modding tasks.

---

## How To: Create a Basic Plugin

### Plugin Class Structure

```java
package com.example.myplugin;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;

public class MyPlugin extends JavaPlugin {

    public MyPlugin(JavaPluginInit init) {
        super(init);  // REQUIRED - must call super
    }

    @Override
    protected void setup() {
        // Early initialization - runs before world is loaded
        // Good for: registering codecs, loading configs
        getLogger().atInfo().log("MyPlugin setup!");
    }

    @Override
    protected void start() {
        // Main initialization - world is available
        // Good for: registering commands, starting systems
        getCommandRegistry().registerCommand(new MyCommand());
        getLogger().atInfo().log("MyPlugin started!");
    }

    @Override
    protected void shutdown() {
        // Cleanup when server stops
        getLogger().atInfo().log("MyPlugin shutdown!");
    }
}
```

### manifest.json

```json
{
  "Group": "MyPluginGroup",
  "Name": "My Plugin",
  "Version": "1.0.0",
  "Description": "A cool Hytale plugin",
  "Main": "com.example.myplugin.MyPlugin",
  "ServerVersion": "*",
  "IncludesAssetPack": false,
  "Authors": [{ "Name": "YourName", "Email": "", "Url": "" }],
  "Website": ""
}
```

**Important:** Does NOT use `onEnable()`/`onDisable()` like Bukkit!

---

## How To: Register Commands with Typed Arguments

### Basic Command

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.Message;
import java.util.concurrent.CompletableFuture;

public class HelloCommand extends AbstractCommand {

    public HelloCommand() {
        super("hello", "Say hello to the world");
        addAliases("hi", "greet");  // Optional aliases
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        ctx.sendMessage(Message.raw("Hello, World!"));
        return CompletableFuture.completedFuture(null);
    }
}
```

### Command with Required Arguments

```java
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;

public class GreetCommand extends AbstractCommand {
    private final RequiredArg<String> nameArg;
    private final RequiredArg<Integer> countArg;

    public GreetCommand() {
        super("greet", "Greet someone multiple times");
        nameArg = withRequiredArg("name", "Person to greet", ArgTypes.STRING);
        countArg = withRequiredArg("count", "Number of times", ArgTypes.INTEGER);
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        String name = ctx.get(nameArg);
        Integer count = ctx.get(countArg);

        for (int i = 0; i < count; i++) {
            ctx.sendMessage(Message.raw("Hello, " + name + "!"));
        }
        return CompletableFuture.completedFuture(null);
    }
}
// Usage: /greet Steve 3
```

### Command with Optional Arguments

```java
import com.hypixel.hytale.server.core.command.system.arguments.system.OptionalArg;

public class TeleportCommand extends AbstractCommand {
    private final RequiredArg<Double> xArg;
    private final RequiredArg<Double> zArg;
    private final OptionalArg<Double> yArg;  // Optional!

    public TeleportCommand() {
        super("tp", "Teleport to coordinates");
        xArg = withRequiredArg("x", "X coordinate", ArgTypes.DOUBLE);
        zArg = withRequiredArg("z", "Z coordinate", ArgTypes.DOUBLE);
        yArg = withOptionalArg("y", "Y coordinate (optional)", ArgTypes.DOUBLE);
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        double x = ctx.get(xArg);
        double z = ctx.get(zArg);
        Double y = ctx.get(yArg);  // null if not provided

        if (y == null) {
            y = 100.0;  // Default value
        }

        ctx.sendMessage(Message.raw("Teleporting to " + x + ", " + y + ", " + z));
        return CompletableFuture.completedFuture(null);
    }
}
```

### Command with Enum Arguments (Tab Completion!)

```java
public enum Difficulty { EASY, MEDIUM, HARD }

public class SetDifficultyCommand extends AbstractCommand {
    private final RequiredArg<Difficulty> difficultyArg;

    public SetDifficultyCommand() {
        super("setdifficulty", "Set game difficulty");
        // ArgTypes.forEnum provides automatic tab completion!
        difficultyArg = withRequiredArg("difficulty", "Difficulty level",
            ArgTypes.forEnum("difficulty", Difficulty.class));
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        Difficulty diff = ctx.get(difficultyArg);  // Type-safe enum value
        ctx.sendMessage(Message.raw("Difficulty set to: " + diff));
        return CompletableFuture.completedFuture(null);
    }
}
```

### Sub-Commands Pattern

```java
public class ConfigCommand extends AbstractCommand {
    public ConfigCommand() {
        super("config", "Manage configuration");

        // Add sub-commands
        addSubCommand(new ReloadSubCommand());
        addSubCommand(new SaveSubCommand());
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        // Called when /config is run with no sub-command
        ctx.sendMessage(Message.raw("Usage: /config <reload|save>"));
        return CompletableFuture.completedFuture(null);
    }

    // Inner class for sub-command
    private static class ReloadSubCommand extends AbstractCommand {
        public ReloadSubCommand() {
            super("reload", "Reload configuration");
        }

        @Override
        protected CompletableFuture<Void> execute(CommandContext ctx) {
            ctx.sendMessage(Message.raw("Configuration reloaded!"));
            return CompletableFuture.completedFuture(null);
        }
    }

    private static class SaveSubCommand extends AbstractCommand {
        public SaveSubCommand() {
            super("save", "Save configuration");
        }

        @Override
        protected CompletableFuture<Void> execute(CommandContext ctx) {
            ctx.sendMessage(Message.raw("Configuration saved!"));
            return CompletableFuture.completedFuture(null);
        }
    }
}
// Usage: /config reload, /config save
```

### Available ArgTypes

| Type | Description |
|------|-------------|
| `ArgTypes.STRING` | Single string |
| `ArgTypes.INTEGER` | Integer number |
| `ArgTypes.DOUBLE` | Decimal number |
| `ArgTypes.FLOAT` | Float number |
| `ArgTypes.BOOLEAN` | true/false |
| `ArgTypes.PLAYER_REF` | Player reference |
| `ArgTypes.ITEM_ASSET` | Item asset |
| `ArgTypes.forEnum("name", Enum.class)` | Enum with tab completion |

---

## How To: Send Colored Chat Messages

**Important:** The `§` color codes do NOT work in Hytale. Use hex colors with `.color()` instead.

### Single Color Message

```java
import com.hypixel.hytale.server.core.Message;

// Green success message
ctx.sendMessage(Message.raw("Success!").color("#55FF55"));

// Red error message
ctx.sendMessage(Message.raw("Error: Something went wrong").color("#FF5555"));

// Orange header
ctx.sendMessage(Message.raw("=== My Plugin ===").color("#FF9900"));
```

### Multi-Color Messages

```java
// Label in gray, value in white
ctx.sendMessage(Message.raw("Health: ").color("#AAAAAA")
    .insert(Message.raw("100").color("#FFFFFF")));

// Complex formatting
ctx.sendMessage(Message.raw("[INFO] ").color("#55FFFF")
    .insert(Message.raw("Player ").color("#AAAAAA"))
    .insert(Message.raw("Steve").color("#FFFFFF"))
    .insert(Message.raw(" joined the game").color("#AAAAAA")));
```

### Common Hex Colors

| Color | Hex | Use Case |
|-------|-----|----------|
| Green | `#55FF55` | Success, enabled |
| Red | `#FF5555` | Error, disabled |
| Orange | `#FF9900` | Headers, titles |
| Yellow | `#FFFF55` | Warnings |
| Aqua | `#55FFFF` | Info tags |
| Gray | `#AAAAAA` | Labels, secondary text |
| White | `#FFFFFF` | Primary values |
| Gold | `#FFAA00` | Special items |

### Helper Method Pattern

```java
public class ChatUtil {
    public static void sendSuccess(CommandContext ctx, String message) {
        ctx.sendMessage(Message.raw(message).color("#55FF55"));
    }

    public static void sendError(CommandContext ctx, String message) {
        ctx.sendMessage(Message.raw("Error: ").color("#FF5555")
            .insert(Message.raw(message).color("#FFAAAA")));
    }

    public static void sendInfo(CommandContext ctx, String label, String value) {
        ctx.sendMessage(Message.raw(label + ": ").color("#AAAAAA")
            .insert(Message.raw(value).color("#FFFFFF")));
    }
}

// Usage
ChatUtil.sendSuccess(ctx, "Item spawned!");
ChatUtil.sendError(ctx, "Player not found");
ChatUtil.sendInfo(ctx, "Current Health", "100");
```

---

## How To: Find Entities Near a Player

### Get Player Position

```java
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.universe.world.World;
import com.hypixel.hytale.server.core.modules.entity.component.TransformComponent;
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;

// Cache component type as static field
private static final ComponentType<EntityStore, TransformComponent> TRANSFORM_TYPE =
    TransformComponent.getComponentType();

// In command execute():
Player player = (Player) ctx.sender();
World world = player.getWorld();
Ref<EntityStore> playerRef = ctx.senderAsPlayerRef();
Store<EntityStore> store = world.getEntityStore().getStore();

// Get transform component directly (no reflection!)
TransformComponent transform = store.getComponent(playerRef, TRANSFORM_TYPE);
Vector3d position = transform.getPosition();

double playerX = position.getX();
double playerY = position.getY();
double playerZ = position.getZ();
```

### Iterate All Entities (on WorldThread)

```java
import com.hypixel.hytale.component.ArchetypeChunk;
import com.hypixel.hytale.component.CommandBuffer;
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.server.core.modules.entity.component.ModelComponent;
import com.hypixel.hytale.server.core.modules.entity.component.TransformComponent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;

// Cache component types as static fields
private static final ComponentType<EntityStore, ModelComponent> MODEL_TYPE =
    ModelComponent.getComponentType();
private static final ComponentType<EntityStore, TransformComponent> TRANSFORM_TYPE =
    TransformComponent.getComponentType();

public void findNearbyEntities(Player player, double radius) {
    World world = player.getWorld();
    Store<EntityStore> store = world.getEntityStore().getStore();

    // MUST run on WorldThread
    world.execute(() -> {
        store.forEachChunk((ArchetypeChunk<EntityStore> chunk, CommandBuffer<EntityStore> buffer) -> {
            int size = chunk.size();
            for (int i = 0; i < size; i++) {
                try {
                    // Get components directly (no reflection!)
                    ModelComponent modelComp = chunk.getComponent(i, MODEL_TYPE);
                    if (modelComp == null) continue;

                    TransformComponent transformComp = chunk.getComponent(i, TRANSFORM_TYPE);
                    if (transformComp == null) continue;

                    // Extract modelAssetId (still needs reflection for private field)
                    String modelAssetId = extractModelAssetId(modelComp);
                    if (modelAssetId == null) continue;

                    // Get entity ref directly
                    Ref<EntityStore> entityRef = chunk.getReferenceTo(i);

                    // Do something with the entity
                    System.out.println("Found: " + modelAssetId);

                } catch (Exception e) {
                    // Entity may have been despawned, skip it
                }
            }
        });
    });
}

// Helper to extract modelAssetId (model field is private)
private static String extractModelAssetId(ModelComponent modelComp) {
    try {
        Field modelField = ModelComponent.class.getDeclaredField("model");
        modelField.setAccessible(true);
        Object model = modelField.get(modelComp);
        if (model == null) return null;

        // Parse from toString: Model{modelAssetId='Cow', ...}
        String str = model.toString();
        int start = str.indexOf("modelAssetId='") + 14;
        int end = str.indexOf("'", start);
        return str.substring(start, end);
    } catch (Exception e) {
        return null;
    }
}
```

### Filter by Entity Type

```java
// Check if entity is a specific type
public boolean isEntityType(String modelAssetId, String... types) {
    for (String type : types) {
        if (type.equals(modelAssetId)) return true;
    }
    return false;
}

// Usage in iteration:
if (isEntityType(modelAssetId, "Cow", "Pig", "Chicken", "Sheep")) {
    // This is a farm animal
}

if (isEntityType(modelAssetId, "Calf", "Piglet", "Chick", "Lamb")) {
    // This is a baby animal
}
```

---

## How To: Spawn NPCs Programmatically

```java
import com.hypixel.hytale.server.npc.NPCPlugin;
import com.hypixel.hytale.server.core.universe.world.World;

public void spawnNPC(World world, String roleId, double x, double y, double z) {
    // Get role index from role name
    int roleIndex = NPCPlugin.get().getIndex(roleId);

    if (roleIndex < 0) {
        System.out.println("Unknown role: " + roleId);
        return;
    }

    // Create position and rotation vectors
    // You'll need to create Vector3d and Vector3f instances
    Object store = world.getEntityStore().getStore();

    // Spawn the entity
    // Returns Pair<Ref<EntityStore>, NPCEntity>
    var result = NPCPlugin.get().spawnEntity(
        store,
        roleIndex,
        position,      // Vector3d
        rotation,      // Vector3f (can be zero rotation)
        null,          // Model (null for default)
        null           // Callback (optional)
    );

    Object entityRef = result.getFirst();
    Object npcEntity = result.getSecond();

    System.out.println("Spawned " + roleId + " at " + x + ", " + y + ", " + z);
}
```

### Common NPC Role IDs

| Category | Role IDs |
|----------|----------|
| **Farm Adults** | `Cow`, `Pig`, `Chicken`, `Sheep`, `Goat`, `Horse`, `Camel`, `Ram`, `Turkey` |
| **Farm Babies** | `Cow_Calf`, `Pig_Piglet`, `Chicken_Chick`, `Sheep_Lamb`, `Goat_Kid`, `Horse_Foal` |
| **Wild** | `Boar`, `Fox`, `Wolf_Black`, `Bear_Grizzly`, `Rabbit`, `Bunny` |
| **Birds** | `Duck`, `Pigeon`, `Owl_Brown`, `Turkey` |

---

## How To: Play Sounds at a Location

```java
import com.hypixel.hytale.server.core.asset.type.soundevent.config.SoundEvent;
import com.hypixel.hytale.server.core.universe.world.SoundUtil;
import com.hypixel.hytale.protocol.SoundCategory;

public void playSound(World world, String soundId, double x, double y, double z) {
    // Get sound index from asset name
    int soundIndex = SoundEvent.getAssetMap().getIndex(soundId);

    if (soundIndex < 0) {
        System.out.println("Unknown sound: " + soundId);
        return;
    }

    // Get the store for the ComponentAccessor
    Object store = world.getEntityStore().getStore();

    // Play 3D sound at position
    SoundUtil.playSoundEvent3d(
        soundIndex,
        SoundCategory.SFX,  // Sound category
        x, y, z,            // Position
        store               // ComponentAccessor
    );
}
```

### Common Sound IDs

| Sound ID | Description |
|----------|-------------|
| `SFX_Consume_Bread` | Eating/chewing sound |
| `SFX_Cow_Idle` | Cow mooing |
| `SFX_Pig_Idle` | Pig oinking |
| `SFX_Chicken_Idle` | Chicken clucking |
| `SFX_Sheep_Idle` | Sheep baaing |
| `SFX_Shears_Activate` | Shearing sound |
| `SFX_Item_Pickup` | Item pickup |

### Sound Categories

| Category | Use Case |
|----------|----------|
| `SoundCategory.SFX` | Sound effects |
| `SoundCategory.MUSIC` | Background music |
| `SoundCategory.AMBIENT` | Ambient/environmental |

---

## How To: Modify Player Inventory

### Get Player Inventory

```java
import com.hypixel.hytale.server.core.entity.LivingEntity;
import com.hypixel.hytale.server.core.inventory.Inventory;
import com.hypixel.hytale.server.core.inventory.container.ItemContainer;

Player player = (Player) ctx.sender();
Inventory inventory = ((LivingEntity) player).getInventory();

// Inventory sections
ItemContainer hotbar = inventory.getHotbar();      // Main hand slots
ItemContainer storage = inventory.getStorage();    // Backpack
ItemContainer armor = inventory.getArmor();        // Armor slots
ItemContainer utility = inventory.getUtility();    // Utility items
ItemContainer tools = inventory.getTools();        // Tool slots
```

### Get Currently Held Item

```java
byte activeSlot = inventory.getActiveHotbarSlot();
ItemStack itemInHand = inventory.getItemInHand();

if (itemInHand != null && !itemInHand.isEmpty()) {
    String itemId = itemInHand.getItemId();
    int quantity = itemInHand.getQuantity();
    ctx.sendMessage(Message.raw("Holding: " + itemId + " x" + quantity));
}
```

### Consume Item from Hand

```java
public void consumeHeldItem(Player player, int amount) {
    Inventory inventory = ((LivingEntity) player).getInventory();
    byte activeSlot = inventory.getActiveHotbarSlot();

    // Remove items from the active hotbar slot
    inventory.getHotbar().removeItemStackFromSlot((short) activeSlot, amount);

    // IMPORTANT: Mark inventory as changed and sync to client
    inventory.markChanged();
    player.sendInventory();
}
```

### Check If Player Has Item

```java
public boolean hasItem(Player player, String itemId, int minQuantity) {
    Inventory inventory = ((LivingEntity) player).getInventory();
    ItemStack item = inventory.getItemInHand();

    if (item != null && !item.isEmpty()) {
        return item.getItemId().equals(itemId) && item.getQuantity() >= minQuantity;
    }
    return false;
}

// Usage
if (hasItem(player, "Plant_Crop_Wheat_Item", 1)) {
    consumeHeldItem(player, 1);
    // Do something
}
```

---

## How To: Create a Periodic Tick System

### Using ScheduledExecutorService

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class MyPlugin extends JavaPlugin {
    private ScheduledExecutorService scheduler;

    @Override
    protected void start() {
        scheduler = Executors.newSingleThreadScheduledExecutor();

        // Run every 5 seconds
        scheduler.scheduleAtFixedRate(
            this::onTick,
            0,                  // Initial delay
            5,                  // Period
            TimeUnit.SECONDS
        );
    }

    private void onTick() {
        try {
            // Your periodic logic here
            getLogger().atInfo().log("Tick!");

            // If you need to access entities, use world.execute()
            World world = Universe.get().getDefaultWorld();
            if (world != null) {
                world.execute(() -> {
                    // Entity operations here (on WorldThread)
                });
            }
        } catch (Exception e) {
            getLogger().atSevere().log("Tick error: %s", e.getMessage());
        }
    }

    @Override
    protected void shutdown() {
        if (scheduler != null) {
            scheduler.shutdown();
            try {
                scheduler.awaitTermination(5, TimeUnit.SECONDS);
            } catch (InterruptedException e) {
                scheduler.shutdownNow();
            }
        }
    }
}
```

### Different Tick Intervals

```java
// Fast tick (every 250ms / 4 times per second)
scheduler.scheduleAtFixedRate(this::fastTick, 0, 250, TimeUnit.MILLISECONDS);

// Normal tick (every second)
scheduler.scheduleAtFixedRate(this::normalTick, 0, 1, TimeUnit.SECONDS);

// Slow tick (every 30 seconds)
scheduler.scheduleAtFixedRate(this::slowTick, 0, 30, TimeUnit.SECONDS);

// Minute tick (every minute)
scheduler.scheduleAtFixedRate(this::minuteTick, 0, 1, TimeUnit.MINUTES);
```

---

## How To: Get Entity Position and Transform

```java
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.server.core.modules.entity.component.TransformComponent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import org.joml.Vector3d;

public class EntityUtil {

    // Cache component type as static field
    private static final ComponentType<EntityStore, TransformComponent> TRANSFORM_TYPE =
        TransformComponent.getComponentType();

    public static double[] getEntityPosition(Store<EntityStore> store, Ref<EntityStore> entityRef) {
        try {
            TransformComponent transform = store.getComponent(entityRef, TRANSFORM_TYPE);
            if (transform == null) return null;

            Vector3d position = transform.getPosition();
            return new double[] { position.x, position.y, position.z };
        } catch (Exception e) {
            return null;
        }
    }

    public static double getDistance(double[] pos1, double[] pos2) {
        double dx = pos1[0] - pos2[0];
        double dy = pos1[1] - pos2[1];
        double dz = pos1[2] - pos2[2];
        return Math.sqrt(dx * dx + dy * dy + dz * dz);
    }

    public static double getHorizontalDistance(double[] pos1, double[] pos2) {
        double dx = pos1[0] - pos2[0];
        double dz = pos1[2] - pos2[2];
        return Math.sqrt(dx * dx + dz * dz);
    }
}
```

---

## How To: Listen to Events

### Register Event Listeners

```java
@Override
protected void start() {
    // Simple event (Void-keyed)
    getEventRegistry().register(PlayerConnectEvent.class, this::onPlayerConnect);
    getEventRegistry().register(PlayerDisconnectEvent.class, this::onPlayerDisconnect);

    // With priority
    getEventRegistry().register(
        EventPriority.EARLY,
        PlayerChatEvent.class,
        this::onPlayerChat
    );

    // Global events (String-keyed like PlayerInteractEvent)
    getEventRegistry().registerGlobal(PlayerInteractEvent.class, this::onInteract);
}

private void onPlayerConnect(PlayerConnectEvent event) {
    Player player = event.getPlayer();
    getLogger().atInfo().log("Player connected: %s", player.getName());
}

private void onPlayerDisconnect(PlayerDisconnectEvent event) {
    Player player = event.getPlayer();
    getLogger().atInfo().log("Player disconnected: %s", player.getName());
}

private void onPlayerChat(PlayerChatEvent event) {
    String message = event.getMessage();
    Player player = event.getPlayer();

    // Cancel the event if needed
    if (message.contains("badword")) {
        event.setCancelled(true);
    }
}
```

### Event Priority

| Priority | Value | Use Case |
|----------|-------|----------|
| `FIRST` | -21844 | Security checks, validation |
| `EARLY` | -10922 | Data transformation |
| `NORMAL` | 0 | Default handling |
| `LATE` | 10922 | Logging, analytics |
| `LAST` | 21844 | Cleanup, fallback |

### PlayerMouseButtonEvent (NOT Recommended for Entity Clicks)

> **WARNING:** `PlayerMouseButtonEvent` does NOT reliably fire for entity interactions during normal gameplay. It only works when the mouse cursor is unlocked (in menus). For entity interactions, use the **Interaction System** instead (see [06-interactions.md](api/06-interactions.md)).

```java
// This does NOT work reliably for entity clicks during gameplay!
// Use SimpleInteraction + Interactions component instead.

getEventRegistry().register(PlayerMouseButtonEvent.class, event -> {
    // Only works in menu/unlocked-cursor situations
    Player player = event.getPlayer();
    MouseButtonType button = event.getMouseButton().mouseButtonType;
    // ...
});
```

### Recommended: Use Interaction System for Entity Clicks

Instead of `PlayerMouseButtonEvent`, create a custom `SimpleInteraction` and attach it to entities:

```java
// 1. Create custom interaction class
public class MyEntityInteraction extends SimpleInteraction {
    public static final BuilderCodec<MyEntityInteraction> CODEC =
        BuilderCodec.builder(MyEntityInteraction.class, MyEntityInteraction::new, SimpleInteraction.CODEC)
            .build();

    @Override
    protected void tick0(boolean firstRun, float time, InteractionType type,
                         InteractionContext context, CooldownHandler cooldownHandler) {
        if (firstRun) {
            Ref<EntityStore> target = context.getTargetEntity();
            ItemStack heldItem = context.getHeldItem();
            // Your custom logic here
        }
        super.tick0(firstRun, time, type, context, cooldownHandler);
    }
}

// 2. Register in setup()
getCodecMapRegistry().register("MyEntityInteraction", MyEntityInteraction.class, MyEntityInteraction.CODEC);

// 3. Create asset files and attach to entities via Interactions component
```

See [06-interactions.md](api/06-interactions.md) for complete details.

---

## How To: Create Verbose Logging Toggle

A common pattern for debugging that can be toggled at runtime.

```java
public class MyPlugin extends JavaPlugin {
    private static boolean verboseLogging = false;

    public static boolean isVerboseLogging() { return verboseLogging; }
    public static void setVerboseLogging(boolean enabled) { verboseLogging = enabled; }

    // Logging helpers
    private void logVerbose(String message) {
        if (verboseLogging) {
            getLogger().atInfo().log("[MyPlugin] " + message);
        }
    }

    private void logWarning(String message) {
        getLogger().atWarning().log("[MyPlugin] " + message);
    }

    private void logError(String message) {
        getLogger().atSevere().log("[MyPlugin] " + message);
    }

    @Override
    protected void start() {
        getCommandRegistry().registerCommand(new ToggleLogsCommand());
    }

    // Toggle command
    public static class ToggleLogsCommand extends AbstractCommand {
        public ToggleLogsCommand() {
            super("myplugindebug", "Toggle verbose logging");
        }

        @Override
        protected CompletableFuture<Void> execute(CommandContext ctx) {
            boolean newState = !MyPlugin.isVerboseLogging();
            MyPlugin.setVerboseLogging(newState);
            ctx.sendMessage(Message.raw("Verbose logging " +
                (newState ? "ENABLED" : "DISABLED")).color(newState ? "#55FF55" : "#FF5555"));
            return CompletableFuture.completedFuture(null);
        }
    }
}
```

---

## How To: Broadcast Debug Logs to In-Game Chat

For easier debugging during development, you can broadcast logs directly to the game chat instead of checking the server console.

```java
import com.hypixel.hytale.server.core.universe.Universe;
import com.hypixel.hytale.server.core.universe.world.World;
import com.hypixel.hytale.server.core.Message;

public class MyPlugin extends JavaPlugin {
    private static MyPlugin instance;
    private static boolean devMode = false;
    private static boolean verboseLogging = false;

    public static MyPlugin getInstance() { return instance; }
    public static boolean isDevMode() { return devMode; }
    public static void setDevMode(boolean enabled) { devMode = enabled; }
    public static boolean isVerboseLogging() { return verboseLogging; }
    public static void setVerboseLogging(boolean enabled) { verboseLogging = enabled; }

    public MyPlugin(JavaPluginInit init) {
        super(init);
        instance = this;
    }

    /** Log verbose message - also broadcasts to chat when devMode is enabled */
    private void logVerbose(String message) {
        if (verboseLogging) {
            getLogger().atInfo().log("[MyPlugin] " + message);
            if (devMode) {
                broadcastToChat(message);
            }
        }
    }

    /** Broadcast a message to all online players in chat */
    private void broadcastToChat(String message) {
        try {
            World world = Universe.get().getDefaultWorld();
            if (world == null) return;

            world.getPlayers().forEach(player -> {
                try {
                    player.sendMessage(Message.raw("[Debug] " + message).color("#AAAAAA"));
                } catch (Exception e) {
                    // Silent
                }
            });
        } catch (Exception e) {
            // Silent
        }
    }

    @Override
    protected void start() {
        getCommandRegistry().registerCommand(new DebugModeCommand());
    }

    /** Toggle command - enables both verbose logging and chat broadcasting */
    public static class DebugModeCommand extends AbstractCommand {
        public DebugModeCommand() {
            super("mydebug", "Toggle in-game chat logging");
        }

        @Override
        protected CompletableFuture<Void> execute(CommandContext ctx) {
            boolean newState = !MyPlugin.isDevMode();
            MyPlugin.setDevMode(newState);
            MyPlugin.setVerboseLogging(newState);  // Enable both together

            String statusColor = newState ? "#55FF55" : "#FF5555";
            String statusText = newState ? "ENABLED" : "DISABLED";
            ctx.sendMessage(Message.raw("Chat logging ").color("#AAAAAA")
                .insert(Message.raw(statusText).color(statusColor)));

            if (newState) {
                ctx.sendMessage(Message.raw("All debug messages will now appear in chat.").color("#FFAA00"));
            }
            return CompletableFuture.completedFuture(null);
        }
    }
}
```

> **Tip:** This is very useful for debugging entity interactions, spawn detection, or any logic that happens away from the console. Just type `/mydebug` in-game to toggle.

---

## How To: Store Per-Entity Data

Since you can't easily add custom components, use a Map keyed by entity identifier.

```java
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.component.Ref;
import com.hypixel.hytale.component.Store;
import com.hypixel.hytale.server.core.entity.UUIDComponent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import java.util.concurrent.ConcurrentHashMap;
import java.util.Map;
import java.util.UUID;

public class EntityDataManager {
    // Cache component type
    private static final ComponentType<EntityStore, UUIDComponent> UUID_TYPE =
        UUIDComponent.getComponentType();

    // Use entity UUID as key (persists across sessions if entity is saved)
    private static final Map<UUID, MyEntityData> entityData = new ConcurrentHashMap<>();

    public static void setData(UUID entityUuid, MyEntityData data) {
        entityData.put(entityUuid, data);
    }

    public static MyEntityData getData(UUID entityUuid) {
        return entityData.get(entityUuid);
    }

    public static void removeData(UUID entityUuid) {
        entityData.remove(entityUuid);
    }

    // Get UUID from entity ref (no reflection needed!)
    public static UUID getEntityUUID(Store<EntityStore> store, Ref<EntityStore> entityRef) {
        try {
            UUIDComponent uuidComp = store.getComponent(entityRef, UUID_TYPE);
            if (uuidComp != null) {
                return uuidComp.getUuid();
            }
        } catch (Exception e) {
            // Entity doesn't have UUID component or was despawned
        }
        return null;
    }
}

// Your custom data class
public class MyEntityData {
    public long lastInteractionTime;
    public int interactionCount;
    public String customState;
    // etc.
}
```

---

## How To: Save/Load Configuration (JSON)

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import java.io.*;
import java.nio.file.*;

public class ConfigManager {
    private static final Gson GSON = new GsonBuilder().setPrettyPrinting().create();
    private final Path configPath;
    private MyConfig config;

    public ConfigManager(Path modsFolder) {
        this.configPath = modsFolder.resolve("myplugin-config.json");
        loadConfig();
    }

    public void loadConfig() {
        if (Files.exists(configPath)) {
            try (Reader reader = Files.newBufferedReader(configPath)) {
                config = GSON.fromJson(reader, MyConfig.class);
            } catch (IOException e) {
                config = new MyConfig();  // Default config
            }
        } else {
            config = new MyConfig();
            saveConfig();  // Create default file
        }
    }

    public void saveConfig() {
        try {
            Files.createDirectories(configPath.getParent());
            try (Writer writer = Files.newBufferedWriter(configPath)) {
                GSON.toJson(config, writer);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public MyConfig getConfig() {
        return config;
    }
}

// Your config class
public class MyConfig {
    public boolean featureEnabled = true;
    public int maxEntities = 100;
    public double spawnRadius = 50.0;
    public List<String> allowedItems = Arrays.asList("Item1", "Item2");
}

// Usage in plugin
@Override
protected void start() {
    Path modsFolder = Paths.get(System.getenv("APPDATA"), "Hytale", "userdata", "saves", "world", "mods");
    ConfigManager configManager = new ConfigManager(modsFolder);

    if (configManager.getConfig().featureEnabled) {
        // Feature is enabled
    }
}
```

---

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

---

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

---

## Interaction System

### Package Structure (actual)
| Class | Full Path |
|-------|-----------|
| `SimpleInstantInteraction` | `com.hypixel.hytale.server.core.modules.interaction.interaction.config.SimpleInstantInteraction` |
| `SimpleInteraction` | `com.hypixel.hytale.server.core.modules.interaction.interaction.config.SimpleInteraction` |
| `Interaction` | `com.hypixel.hytale.server.core.modules.interaction.interaction.config.Interaction` |
| `InteractionContext` | `com.hypixel.hytale.server.core.entity.InteractionContext` |
| `InteractionType` | `com.hypixel.hytale.protocol.InteractionType` |
| `InteractionState` | `com.hypixel.hytale.protocol.InteractionState` |
| `CooldownHandler` | `com.hypixel.hytale.server.core.modules.interaction.interaction.CooldownHandler` |
| `Interactions` (component) | `com.hypixel.hytale.server.core.modules.interaction.Interactions` |

### InteractionType (protocol)
Values: `Primary` (LMB), `Secondary` (RMB), `Use` (E), `Scroll`

### InteractionState (protocol)
Values: `Finished`, `Skip`, `ItemChanged`, `Failed`, `NotFinished`

### InteractionContext Methods
- `getEntity()` → Ref<EntityStore> - The entity performing interaction
- `getTargetEntity()` → Ref<EntityStore> - The target entity
- `getHeldItem()` → ItemStack - Currently held item
- `getTargetBlock()` → BlockPosition
- And many more...

---

## ItemStack

Located at: `com.hypixel.hytale.server.core.inventory.ItemStack`

**Methods:**
- `getItemId()` → String - **NOT** `getType().getId()`
- `getItem()` → Item
- `getQuantity()` → int
- `isEmpty()` → boolean
- `getMetadata()` → BsonDocument

---

## World/Universe

### Package Structure (actual)
| Class | Full Path |
|-------|-----------|
| `Universe` | `com.hypixel.hytale.server.core.universe.Universe` |
| `World` | `com.hypixel.hytale.server.core.universe.world.World` |
| `EntityStore` | `com.hypixel.hytale.server.core.universe.world.storage.EntityStore` |

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

### Store (inner ECS store)
**Methods:**
- `getEntityCount()` → int - Total entity count
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
- `collectArchetypeChunkData()` → ArchetypeChunkData[] - **USEFUL: Returns array for iteration!**

**Iteration methods (require WorldThread):**
- `forEachChunk(BiPredicate)` → boolean
- `forEachChunk(BiConsumer)` → void
- `forEachEntityParallel(IntBiObjectConsumer)` → void - **Requires WorldThread!**

---

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
- `get()` → null (**doesn't work** for player entity access!)

**Important:** Use `ctx.sender()` directly to get the Player object, not `senderAsPlayerRef().get()`.

---

## Events

**PlayerInteractEvent is deprecated.** Alternatives:
- `PlayerMouseButtonEvent` - Only fires with unlocked mouse (menus)
- `UseBlockEvent.Pre/Post` - For block interactions only

For entity interactions during gameplay, use the **Interaction system** (attach interactions to entity's Interactions component).

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
| 72 | InteractionManager | `com.hypixel.hytale.server.core.entity` |
| 73 | Interactions | `com.hypixel.hytale.server.core.modules.interaction` |
| 74 | ChainingInteraction$Data | (chaining interaction data) |
| 131 | SpawnMarkerEntity | `com.hypixel.hytale.server.spawning.spawnmarkers` |

**Key components for entity identification:**
- `PrefabCopyableComponent` (60) - Contains the prefab/entity type ID
- `ModelComponent` (41) - Contains the 3D model reference
- `Nameplate` (68) - Display name for entities

### Component Access Pattern
```java
HealthComponent health = store.getComponent(entityRef, healthComponentType);
// Or ensure existence:
HealthComponent health = store.ensureAndGetComponent(entityRef, healthComponentType);
```

### Entity References (`Ref<ECS_TYPE>`)
- Always check `isValid()` before using - references become invalid when entities removed
- Contains: store reference, index, validity status

### NPC/Creature System
- `NPCEntity` is base component for all NPCs
- Creatures are **asset-driven** - defined through asset files, not code
- `Role` class encapsulates complete behavior definitions
- Spawn components: `SpawnBeaconReference`, `SpawnMarkerReference`

**Implication:** Entity types (cow, pig, etc.) are determined by their asset configuration, accessible through `ModelComponent` or `PrefabCopyableComponent`.

### Entity Type Identification

**The `ModelComponent.model` field contains `modelAssetId`** which identifies the entity type:

```java
// ModelComponent has a 'model' field with modelAssetId
// Examples: Frog_Blue, Woodpecker, Duck, NPC_Path_Marker, Player

// Access pattern:
Object modelComp = chunk.getComponent(entityIndex, modelComponentType);
Field modelField = modelComp.getClass().getDeclaredField("model");
modelField.setAccessible(true);
Object model = modelField.get(modelComp);
// model.toString() shows: Model{modelAssetId='Duck', scale=1.0, ...}
```

**Known entity modelAssetIds (discovered in-game):**

| Category | modelAssetId Values |
|----------|---------------------|
| **Farm Animals** | `Cow`, `Calf`, `Pig`, `Piglet`, `Chicken`, `Chick`, `Sheep`, `Lamb`, `Goat`, `Goat_Kid`, `Horse`, `Horse_Foal` |
| **Wild Animals** | `Boar`, `Boar_Piglet`, `Fox`, `Wolf_Black`, `Bear_Grizzly`, `Model_Deer_Stag` |
| **Birds** | `Duck`, `Woodpecker`, `Pigeon`, `Sparrow`, `Finch_Green`, `Owl_Brown`, `Tetrabird` |
| **Small Creatures** | `Bunny`, `Rabbit`, `Squirrel`, `Mouse`, `Frog_Blue`, `Frog_Green` |
| **Fish** | `Minnow`, `Bluegill`, `Trout_Rainbow` |
| **Hostile** | `Spider`, `Skeleton_Fighter` |
| **System** | `Player`, `NPC_Path_Marker`, `NPC_Spawn_Marker` |

### Iterating Entities on WorldThread

Entity iteration **MUST** run on the WorldThread. Use `World.execute(Runnable)`:

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

### Component Indices (on entities)
- **[72]**: `InteractionManager` - `com.hypixel.hytale.server.core.entity.InteractionManager`
- **[73]**: `Interactions` - `com.hypixel.hytale.server.core.modules.interaction.Interactions`
- **[74]**: `ChainingInteraction$Data` - chaining interaction data

### Interactions Class
Located at: `com.hypixel.hytale.server.core.modules.interaction.Interactions`

**Methods:**
- `getComponentType()` → ComponentType - Get the ComponentType for ECS access
- `getInteractions()` → Map - Get all interactions
- `setInteractionId(InteractionType, String)` → void - **Set interaction by type**
- `getInteractionId(InteractionType)` → String - Get interaction ID for type
- `setInteractionHint(String)` → void - Set UI hint text
- `getInteractionHint()` → String
- `clone()` → Component
- `cloneSerializable()` → Component

**Fields:**
- `CODEC` : BuilderCodec
- `interactions` : Map - The interactions map
- `interactionHint` : String
- `isNetworkOutdated` : boolean

### Attaching Custom Interactions to Entities
```java
// 1. Get Interactions component from entity
Class<?> interactionsClass = Class.forName(
    "com.hypixel.hytale.server.core.modules.interaction.Interactions");
Object interactionsType = interactionsClass.getMethod("getComponentType").invoke(null);
Object interactionsComp = store.ensureAndGetComponent(entityRef, interactionsType);

// 2. Get InteractionType enum value
Class<?> interactionTypeClass = Class.forName("com.hypixel.hytale.protocol.InteractionType");
Object secondaryType = null;
for (Object enumVal : interactionTypeClass.getEnumConstants()) {
    if (enumVal.toString().equals("Secondary")) {  // Right-click
        secondaryType = enumVal;
        break;
    }
}

// 3. Set the interaction ID
Method setIntId = interactionsComp.getClass().getMethod(
    "setInteractionId", interactionTypeClass, String.class);
setIntId.invoke(interactionsComp, secondaryType, "YourInteractionId");

// 4. Optionally set hint text (uses localization keys)
Method setHint = interactionsComp.getClass().getMethod("setInteractionHint", String.class);
setHint.invoke(interactionsComp, "server.interactionHints.generic");
```

### InteractionType Values (from protocol)
| Value | Trigger |
|-------|---------|
| `Primary` | Left mouse button |
| `Secondary` | Right mouse button |
| `Use` | E key |
| `Scroll` | Mouse scroll |

---

## RootInteraction System

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

**Asset Paths (confirmed working):**
| Asset Type | Path |
|------------|------|
| RootInteraction | `Server/Item/RootInteractions/{id}.json` |
| Interaction | `Server/Item/Interactions/{id}.json` |

### Registering Custom Interactions

**What works:**
- Setting interaction ID on entities via `setInteractionId(InteractionType, String)` ✓
- Creating RootInteraction assets at `Item/RootInteractions/` ✓
- Creating Interaction assets at `Item/Interactions/` ✓
- Codec registration: `getCodecRegistry(Interaction.CODEC).register("MyType", ...)` ✓

**Known challenges:**
- The `*` prefix mechanism for code-registered interactions is not fully documented
- Custom interactions may require additional registration steps beyond asset files

---

## Useful Patterns

### Safe Ref Usage (Handling Stale References)

Entity refs can become stale when entities despawn. Always wrap ref usage:

```java
Object component = null;
try {
    component = store.getComponent(entityRef, componentType);
} catch (Exception e) {
    Throwable cause = e;
    if (e instanceof java.lang.reflect.InvocationTargetException) {
        cause = ((java.lang.reflect.InvocationTargetException) e).getTargetException();
    }

    if (cause instanceof IllegalStateException &&
        cause.getMessage() != null &&
        cause.getMessage().contains("Invalid entity")) {
        // Entity was despawned - clean up tracking data
        return;
    }
    throw e;
}
```

### Getting Model Asset ID from Entity

```java
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.server.core.modules.entity.component.ModelComponent;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;

// Cache component type as static field
private static final ComponentType<EntityStore, ModelComponent> MODEL_TYPE =
    ModelComponent.getComponentType();

// Get component from store (no reflection for component type!)
ModelComponent modelComp = store.getComponent(entityRef, MODEL_TYPE);

// Extract modelAssetId (model field is private, so still needs reflection)
Field modelField = ModelComponent.class.getDeclaredField("model");
modelField.setAccessible(true);
Object model = modelField.get(modelComp);

// Parse from toString: Model{modelAssetId='Cow', ...}
String modelStr = model.toString();
int start = modelStr.indexOf("modelAssetId='") + 14;
int end = modelStr.indexOf("'", start);
String modelAssetId = modelStr.substring(start, end);  // e.g., "Cow", "Sheep"
```

---

## Key Class Paths Summary

| Class | Full Path |
|-------|-----------|
| `JavaPlugin` | `com.hypixel.hytale.server.core.plugin.JavaPlugin` |
| `AbstractCommand` | `com.hypixel.hytale.server.core.command.system.AbstractCommand` |
| `CommandContext` | `com.hypixel.hytale.server.core.command.system.CommandContext` |
| `Message` | `com.hypixel.hytale.server.core.Message` |
| `Player` | `com.hypixel.hytale.server.core.entity.entities.Player` |
| `World` | `com.hypixel.hytale.server.core.universe.world.World` |
| `Universe` | `com.hypixel.hytale.server.core.universe.Universe` |
| `EntityStore` | `com.hypixel.hytale.server.core.universe.world.storage.EntityStore` |
| `Ref` | `com.hypixel.hytale.component.Ref` |
| `Store` | `com.hypixel.hytale.component.Store` |
| `Archetype` | `com.hypixel.hytale.component.Archetype` |
| `ArchetypeChunk` | `com.hypixel.hytale.component.ArchetypeChunk` |
| `ModelComponent` | `com.hypixel.hytale.server.core.modules.entity.component.ModelComponent` |
| `TransformComponent` | `com.hypixel.hytale.server.core.modules.entity.component.TransformComponent` |
| `Interactions` | `com.hypixel.hytale.server.core.modules.interaction.Interactions` |
| `InteractionType` | `com.hypixel.hytale.protocol.InteractionType` |
| `ItemStack` | `com.hypixel.hytale.server.core.inventory.ItemStack` |
| `NPCPlugin` | `com.hypixel.hytale.server.npc.NPCPlugin` |

---

## Contributing

Found something new? These discoveries came from runtime exploration using reflection. If you discover additional API details, consider sharing them with the modding community!
