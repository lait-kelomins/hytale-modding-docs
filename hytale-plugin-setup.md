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

### 2. settings.gradle

```groovy
rootProject.name = 'my-hytale-plugin'
```

### 3. gradle.properties

```properties
org.gradle.jvmargs=-Xmx2G
org.gradle.parallel=true
org.gradle.caching=true
```

### 4. build.gradle

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

    // Optional: common libraries
    implementation 'com.google.guava:guava:32.1.3-jre'
    implementation 'com.google.code.gson:gson:2.10.1'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.1'
}

shadowJar {
    archiveClassifier.set('')

    // Exclude server classes (they're provided at runtime)
    dependencies {
        exclude(dependency { it.moduleGroup == 'com.hypixel.hytale' })
    }
}

build {
    dependsOn shadowJar
}

test {
    useJUnitPlatform()
}
```

### 5. manifest.json

```json
{
  "Group": "YourOrg",
  "Name": "MyPlugin",
  "Version": "1.0.0",
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

### 6. Main Plugin Class

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
        // Register commands, events, initialize resources
        getLogger().atInfo().log("Plugin setup complete!");
    }

    @Override
    protected void start() {
        // Load configs, connect databases, start tasks
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

// Register in setup():
getCommandRegistry().registerCommand(new HelloCommand());
```

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

Copy the JAR to your server's mods folder:

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
