# Hytale Plugin Setup Guide

A quick start guide for creating Hytale server plugins.

## Prerequisites

- **Java 21 JDK** (required - HytaleServer.jar is compiled with Java 21)
- **Gradle 9.x**
- **HytaleServer.jar** - The server JAR (not redistributable, place in `libs/`)
- IDE (IntelliJ IDEA recommended)

---

## Quick Start

### 1. Create Project Structure

```
my-hytale-plugin/
├── src/main/java/com/yourname/pluginname/
│   └── MyPlugin.java
├── src/main/resources/
│   └── manifest.json
├── libs/
│   └── HytaleServer.jar    # compileOnly dependency
├── build.gradle
├── settings.gradle
└── gradle.properties
```

### 2. Initialize Gradle Wrapper

Run this once in your project folder to create `gradlew`/`gradlew.bat`:

```bash
gradle wrapper --gradle-version 9.0
```

### 3. settings.gradle

```groovy
rootProject.name = 'my-hytale-plugin'
```

### 4. gradle.properties

```properties
org.gradle.jvmargs=-Xmx2G
org.gradle.parallel=true
org.gradle.caching=true
```

### 5. build.gradle

```groovy
plugins {
    id 'java'
    id 'com.gradleup.shadow' version '8.3.0'
}

group = 'com.yourname'
version = '1.0.0'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // HytaleServer.jar must be in libs/ folder (compileOnly - not bundled)
    compileOnly files('libs/HytaleServer.jar')

    // Note: Guava and Gson are provided by the server at runtime
    // Only add dependencies if you need something not already included

    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.1'
}

// Replace ${version} in manifest.json with actual version
processResources {
    filesMatching('manifest.json') {
        expand('version': project.version)
    }
}

shadowJar {
    archiveClassifier.set('')
}

build {
    dependsOn shadowJar
}

test {
    useJUnitPlatform()
}
```

### 6. manifest.json

```json
{
  "Group": "YourOrg",
  "Name": "MyPlugin",
  "Version": "${version}",
  "Description": "A custom Hytale plugin",
  "Main": "com.yourname.pluginname.MyPlugin",
  "ServerVersion": "*",
  "IncludesAssetPack": false,
  "Authors": [
    { "Name": "YourName", "Email": "", "Url": "" }
  ],
  "Website": ""
}
```

> **Note:** `${version}` is replaced by Gradle's `processResources` task with the version from `build.gradle`.

### 7. Main Plugin Class

```java
package com.yourname.pluginname;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import javax.annotation.Nonnull;

public class MyPlugin extends JavaPlugin {

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);  // REQUIRED
    }

    @Override
    protected void setup() {
        // Early initialization - register event listeners
        getLogger().atInfo().log("Plugin setup complete!");
    }

    @Override
    protected void start() {
        // Register commands, load configs, start tasks
        getLogger().atInfo().log("Plugin started!");
    }

    @Override
    protected void shutdown() {
        // Save data, close connections, cleanup
        getLogger().atInfo().log("Plugin shutting down!");
    }
}
```

**Important:** Does NOT use `onLoad()`, `onEnable()`, `onDisable()` like Bukkit/Spigot!

---

## Adding Features

### Event Listener

```java
// In setup() method:
getEventRegistry().register(PlayerConnectEvent.class, event -> {
    Player player = event.getPlayer();
    getLogger().atInfo().log("Player connected: %s", player.getName());
});
```

### Command

```java
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.Message;
import java.util.concurrent.CompletableFuture;

public class HelloCommand extends AbstractCommand {

    public HelloCommand() {
        super("hello", "Says hello");
    }

    @Override
    protected CompletableFuture<Void> execute(CommandContext ctx) {
        ctx.sendMessage(Message.raw("Hello, World!"));
        return CompletableFuture.completedFuture(null);
    }
}

// Register in start():
getCommandRegistry().registerCommand(new HelloCommand());
```

---

## Asset Packs

To include custom assets (sounds, particles, interactions, etc.), set `"IncludesAssetPack": true` in your manifest and add assets to:

```
src/main/resources/
├── Common/           # Shared assets (sounds, textures)
│   └── Sounds/
│       └── MySound.ogg
└── Server/           # Server-side assets
    ├── Audio/
    │   └── SoundEvents/
    │       └── SFX_MySound.json
    ├── Item/
    │   ├── Interactions/
    │   └── RootInteractions/
    └── Particles/
```

See [How To: Add Custom Sounds](api/api-empirical-how-tos.md#how-to-add-custom-sounds) for details.

---

## Build & Deploy

### Windows

```cmd
set JAVA_HOME=C:\path\to\jdk-21
gradlew.bat build -x test
```

### Linux/Mac

```bash
JAVA_HOME=/path/to/jdk21 ./gradlew build -x test
```

### Deploy

Copy the JAR from `build/libs/` to your server's mods folder:

```
%APPDATA%/Hytale/userdata/saves/<world_name>/mods/
```

---

## Logging

```java
getLogger().atInfo().log("Info message");
getLogger().atInfo().log("Formatted: %s", value);
getLogger().atWarning().log("Warning message");
getLogger().atSevere().log("Error message");
```

---

## Next Steps

- [Plugin System](api/01-plugin-system.md) - Detailed plugin lifecycle
- [Commands](api/04-commands.md) - Advanced command patterns
- [Events](api/05-events.md) - Event handling system
- [ECS System](api/02-ecs-system.md) - Entity Component System
- [How-To Examples](api/api-empirical-how-tos.md) - Practical code examples
