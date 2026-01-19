# Hytale Command System

> **Source:** https://hytale-docs.com/docs/api/server-internals/commands

---

## Overview

The command system uses a type-safe argument system with inheritance-based command types.

**Key Packages:**
```
com.hypixel.hytale.server.core.command.system/           # Core classes
com.hypixel.hytale.server.core.command.system.arguments/ # Argument types
com.hypixel.hytale.server.core.command.commands/         # Built-in commands
```

---

## Base Classes & Hierarchy

| Class | Purpose |
|-------|---------|
| `AbstractCommand` | Root base class |
| `CommandBase` | Synchronous commands |
| `AbstractAsyncCommand` | Asynchronous operations |
| `AbstractPlayerCommand` | Player-restricted commands |
| `AbstractWorldCommand` | World-context commands |
| `AbstractEntityCommand` | Entity operations |
| `AbstractCommandCollection` | Subcommands only (no direct execution) |

---

## Creating Commands

### Simple Synchronous Command

```java
import com.hypixel.hytale.server.core.command.system.CommandBase;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.Message;

public class MyCommand extends CommandBase {

    public MyCommand() {
        super("mycommand", "desc.key");
    }

    @Override
    protected void executeSync(CommandContext context) {
        context.sendMessage(Message.raw("Hello!"));
    }
}
```

### Asynchronous Command

```java
import com.hypixel.hytale.server.core.command.system.AbstractAsyncCommand;
import java.util.concurrent.CompletableFuture;

public class MyAsyncCommand extends AbstractAsyncCommand {

    public MyAsyncCommand() {
        super("myasync", "desc.key");
    }

    @Override
    protected CompletableFuture<Void> executeAsync(CommandContext context) {
        return CompletableFuture.runAsync(() -> {
            // Long-running operation
            context.sendMessage(Message.raw("Done!"));
        });
    }
}
```

### Player-Restricted Command

```java
import com.hypixel.hytale.server.core.command.system.AbstractPlayerCommand;

public class MyPlayerCommand extends AbstractPlayerCommand {

    public MyPlayerCommand() {
        super("myplayercmd", "desc.key");
    }

    @Override
    protected void execute(CommandContext ctx, Store<EntityStore> store,
            Ref<EntityStore> ref, PlayerRef playerRef, World world) {
        // Access player data and world
        playerRef.getPlayer().sendMessage(Message.raw("Hello player!"));
    }
}
```

---

## Argument System

Arguments are type-safe and come in four varieties:

| Type | Syntax | Description |
|------|--------|-------------|
| `RequiredArg` | `<name>` | Must be provided |
| `OptionalArg` | `--name=value` | Can be omitted |
| `DefaultArg` | `--name=value` | Has fallback value |
| `FlagArg` | `--name` | Boolean toggle |

### Defining Arguments

```java
public class MyCommand extends CommandBase {

    private final RequiredArg<String> nameArg =
        withRequiredArg("name", "desc.name", ArgTypes.STRING);

    private final OptionalArg<Integer> countArg =
        withOptionalArg("count", "desc.count", ArgTypes.INTEGER);

    private final DefaultArg<Double> speedArg =
        withDefaultArg("speed", "desc.speed", ArgTypes.DOUBLE, 1.0);

    private final FlagArg verboseFlag =
        withFlagArg("verbose", "desc.verbose");

    public MyCommand() {
        super("mycommand", "desc.key");
    }

    @Override
    protected void executeSync(CommandContext ctx) {
        String name = nameArg.get(ctx);

        if (countArg.provided(ctx)) {
            Integer count = countArg.get(ctx);
        }

        Double speed = speedArg.get(ctx); // Uses default if not provided

        Boolean verbose = verboseFlag.get(ctx);
    }
}
```

### Common Argument Types

| Type | Description |
|------|-------------|
| `ArgTypes.STRING` | Text string |
| `ArgTypes.INTEGER` | Whole number |
| `ArgTypes.DOUBLE` | Decimal number |
| `ArgTypes.BOOLEAN` | True/false |
| `ArgTypes.PLAYER_REF` | Player reference |
| `ArgTypes.WORLD` | World reference |
| `ArgTypes.ITEM_ASSET` | Item type |
| `ArgTypes.BLOCK_TYPE_KEY` | Block type |
| `ArgTypes.GAME_MODE` | Game mode |
| `ArgTypes.RELATIVE_POSITION` | Position with ~ support |
| `ArgTypes.ROTATION` | Rotation values |
| `ArgTypes.ENTITY_ID` | Entity identifier |

---

## Command Context

`CommandContext` provides execution context:

```java
CommandContext ctx;

// Sender info
CommandSender sender = ctx.sender();
boolean isPlayer = ctx.isPlayer();

// Output
ctx.sendMessage(Message.raw("Text"));

// Argument retrieval (via argument objects)
String value = myArg.get(ctx);
boolean provided = myOptionalArg.provided(ctx);
```

---

## Command Registration

### Plugin Commands (in setup())

```java
@Override
protected void setup() {
    getCommandRegistry().registerCommand(new MyCommand());
    getCommandRegistry().registerCommand(new MyPlayerCommand());
}
```

### System Commands

```java
CommandManager.get().registerSystemCommand(new MySystemCommand());
```

---

## Permissions

```java
public class MyCommand extends CommandBase {

    public MyCommand() {
        super("mycommand", "desc.key");

        // Set required permission
        requirePermission("myplugin.command.mycommand");

        // Or use built-in format
        requirePermission(HytalePermissions.fromCommand("mycommand"));
    }
}
```

**Auto-generated format:** `hytale.system.command.<name>` for system commands.

---

## Built-in Commands

### Player Commands

| Command | Class | Purpose |
|---------|-------|---------|
| `/gamemode` | `GameModeCommand` | Change game mode |
| `/give` | `GiveCommand` | Give items |
| `/kill` | `KillCommand` | Kill player |
| `/damage` | `DamageCommand` | Apply damage |
| `/respawn` | `PlayerRespawnCommand` | Respawn player |

### Entity Commands

| Command | Class | Purpose |
|---------|-------|---------|
| `/entity` | `EntityCommand` | Entity management |
| `/entity clone` | `EntityCloneCommand` | Clone entity |
| `/entity remove` | `EntityRemoveCommand` | Remove entity |
| `/entity effect` | `EntityEffectCommand` | Apply effects |

### Plugin Commands

| Command | Description |
|---------|-------------|
| `/plugin list` | List loaded plugins |
| `/plugin load <Group:Name>` | Load plugin |
| `/plugin unload <Group:Name>` | Unload plugin |
| `/plugin reload <Group:Name>` | Reload plugin |
| `/plugin manage` | Open management UI |

### Inventory Commands

| Class | Purpose |
|-------|---------|
| `GiveCommand` | Give items |
| `GiveArmorCommand` | Give armor set |
| `InventoryCommand` | Inventory management |
| `InventoryClearCommand` | Clear inventory |
| `InventorySeeCommand` | View inventory |

---

## CommandSender Interface

Represents command executors (console or player):

```java
public interface CommandSender {
    String getDisplayName();
    UUID getUuid();
    boolean hasPermission(String permission);
    void sendMessage(Message message);
}
```

---

## Example: Complete Command

```java
package com.example.myplugin.commands;

import com.hypixel.hytale.server.core.command.system.CommandBase;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.Message;

public class TeleportCommand extends CommandBase {

    private final RequiredArg<String> playerArg =
        withRequiredArg("player", "Player name", ArgTypes.STRING);

    public TeleportCommand() {
        super("tpto", "Teleport to a player");
        requirePermission("myplugin.teleport");
    }

    @Override
    protected void executeSync(CommandContext ctx) {
        String playerName = playerArg.get(ctx);

        ctx.sendMessage(Message.raw("Teleporting to " + playerName + "..."));

        // Teleport logic here

        ctx.sendMessage(Message.raw("Teleported!"));
    }
}
```

### Registration

```java
@Override
protected void setup() {
    getCommandRegistry().registerCommand(new TeleportCommand());
}
```

---

## Related Files

- [01-plugin-system.md](01-plugin-system.md) - Plugin lifecycle and registration
- [05-events.md](05-events.md) - Event system
