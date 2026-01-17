# Hytale Plugin System

> **Source:** https://hytale-docs.com/docs/api/server-internals/plugins

---

## Overview

Hytale plugins extend `JavaPlugin` and are managed by the `PluginManager`.

**Key Classes:**
```
com.hypixel.hytale.server.core.plugin.JavaPlugin        # Base class
com.hypixel.hytale.server.core.plugin.JavaPluginInit    # Constructor parameter
com.hypixel.hytale.server.core.plugin.PluginManager     # Manages plugins
com.hypixel.hytale.server.core.plugin.PluginBase        # Abstract base
com.hypixel.hytale.common.plugin.PluginManifest         # Plugin metadata
com.hypixel.hytale.server.core.plugin.PluginClassLoader # Class loading
```

---

## JavaPlugin

Base class for all Hytale plugins. Must accept `JavaPluginInit` in constructor.

```java
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import javax.annotation.Nonnull;

public class MyPlugin extends JavaPlugin {

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        // Register commands, events, initialize resources
    }

    @Override
    protected void start() {
        // Load configs, connect databases, start tasks
    }

    @Override
    protected void shutdown() {
        // Save data, close connections, cleanup
    }
}
```

### JavaPluginInit Methods

| Method | Returns | Purpose |
|--------|---------|---------|
| `getPluginManifest()` | PluginManifest | Plugin configuration |
| `getDataDirectory()` | Path | Plugin's data folder |
| `getFile()` | Path | Plugin JAR path |
| `getClassLoader()` | ClassLoader | Plugin's class loader |

---

## Plugin Lifecycle

```
NONE → SETUP → START → ENABLED → SHUTDOWN → DISABLED
```

### Lifecycle Methods

| Method | When Called | What To Do |
|--------|-------------|------------|
| `preLoad()` | Before setup (optional) | Async config loading |
| `setup()` | After dependencies resolved | **Register events, commands, systems** |
| `start()` | All plugins set up | Load configs, connect databases, start tasks |
| `shutdown()` | Server stopping | Save data, cleanup resources |

**IMPORTANT:** Events and commands MUST be registered in `setup()`, not `start()`!

---

## Plugin Manifest

File: `src/main/resources/manifest.json`

```json
{
  "Group": "OrganizationId",
  "Name": "PluginName",
  "Version": "1.0.0",
  "Description": "Brief description",
  "Authors": [
    {"Name": "Author", "Email": "email@example.com", "Url": "https://example.com"}
  ],
  "Website": "https://plugin-website.com",
  "Main": "com.example.PluginClassName",
  "ServerVersion": ">=0.1.0",
  "Dependencies": {"Group:Plugin": ">=1.0.0"},
  "OptionalDependencies": {"Group:Plugin": ">=1.0.0"},
  "LoadBefore": {"Group:Plugin": "*"},
  "DisabledByDefault": false,
  "IncludesAssetPack": true,
  "SubPlugins": [
    {"Name": "SubName", "Main": "com.example.SubPlugin"}
  ]
}
```

### Manifest Fields

| Field | Required | Description |
|-------|----------|-------------|
| `Group` | Yes | Organization identifier |
| `Name` | Yes | Plugin name |
| `Version` | Yes | SemVer version string |
| `Main` | Yes | Fully qualified main class |
| `Description` | No | Plugin description |
| `Authors` | No | List of author objects |
| `ServerVersion` | No | Required server version range |
| `Dependencies` | No | Required plugin dependencies |
| `IncludesAssetPack` | No | Whether plugin has assets |

### Version Range Syntax

- `*` - Any version
- `1.0.0` - Exact match
- `>=1.0.0` - 1.0.0 or higher
- `>=1.0.0 <2.0.0` - Range

---

## Registry Access

### CommandRegistry
```java
getCommandRegistry().registerCommand(new MyCommand());
```

### EventRegistry
```java
// Standard registration
getEventRegistry().register(EventClass.class, this::handler);

// With priority
getEventRegistry().register(EventPriority.HIGH, EventClass.class, handler);

// Global (all events of type, ignoring key)
getEventRegistry().registerGlobal(KeyedEvent.class, handler);

// Async events
getEventRegistry().registerAsync(AsyncEvent.class, futureHandler);
```

### TaskRegistry
```java
getTaskRegistry().registerTask(completableFuture);
getTaskRegistry().registerTask(scheduledFuture);
```

### EntityStoreRegistry (ECS)
```java
getEntityStoreRegistry().registerSystem(new MySystem());
```

### Other Registries
- `getBlockStateRegistry()`
- `getEntityRegistry()`
- `getClientFeatureRegistry()`
- `getAssetRegistry()`
- `getChunkStoreRegistry()`
- `getCodecMapRegistry()`

---

## PluginManager

```java
// Get manager
PluginManager manager = PluginManager.get();

// Query plugins
manager.getPlugins();
manager.getPlugin(new PluginIdentifier("Group", "Name"));
manager.hasPlugin(identifier, SemverRange);
manager.getAvailablePlugins();

// Manage plugins
manager.load(identifier);
manager.unload(identifier);
manager.reload(identifier);
```

### Plugin Identifier Format
```
Group:Name
```
Example: `MyOrg:MyPlugin`

---

## Server Access

```java
HytaleServer.get();              // Server instance
server.getEventBus();            // Event bus
server.getConfig();              // Server configuration
getLogger();                     // Plugin logger
getDataDirectory();              // Plugin data folder
getBasePermission();             // Base permission (group.name)
```

---

## Console Commands

| Command | Description |
|---------|-------------|
| `/plugin list` | List loaded plugins |
| `/plugin load <Group:Name>` | Load plugin |
| `/plugin unload <Group:Name>` | Unload plugin |
| `/plugin reload <Group:Name>` | Reload plugin |
| `/plugin manage` | Open management UI |

---

## Example Plugin

```java
package com.example.myplugin;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import javax.annotation.Nonnull;

public class MyPlugin extends JavaPlugin {

    private static MyPlugin instance;

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
        instance = this;
    }

    @Override
    protected void setup() {
        getLogger().atInfo().log("Setting up...");

        // Register events
        getEventRegistry().register(UseBlockEvent.Pre.class, this::onUseBlock);

        // Register commands
        getCommandRegistry().registerCommand(new MyCommand());

        // Register ECS systems
        getEntityStoreRegistry().registerSystem(new MySystem());
    }

    @Override
    protected void start() {
        getLogger().atInfo().log("Starting...");
        // Load configs, start scheduled tasks
    }

    @Override
    protected void shutdown() {
        getLogger().atInfo().log("Shutting down...");
        // Cleanup
    }

    public static MyPlugin getInstance() {
        return instance;
    }
}
```

---

## Related Files

- [02-ecs-system.md](./02-ecs-system.md) - ECS components and systems
- [04-commands.md](./04-commands.md) - Command registration
- [05-events.md](./05-events.md) - Event system
