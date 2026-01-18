# Hytale Plugin Setup Guide

A quick start guide for creating Hytale server plugins.

## Prerequisites

- **Java 21 JDK** (required - HytaleServer.jar is compiled with Java 21)
- **Gradle 9.x**
- **HytaleServer.jar** - The server JAR (not redistributable, place in `libs/`)
- IDE (IntelliJ IDEA recommended)

---

## Installation

### Install Java 21 JDK

#### Windows

**Option 1: Package Manager (recommended)**

```powershell
# Chocolatey
choco install microsoft-openjdk21

# Scoop
scoop bucket add java
scoop install openjdk21

# winget
winget install Microsoft.OpenJDK.21
```

**Option 2: Manual Install**

1. Download from [Microsoft OpenJDK](https://learn.microsoft.com/en-us/java/openjdk/download#openjdk-21) or [Adoptium](https://adoptium.net/temurin/releases/?version=21)
2. Run the installer
3. Set `JAVA_HOME` environment variable:
   ```cmd
   setx JAVA_HOME "C:\Program Files\Microsoft\jdk-21.0.x-hotspot"
   ```

**Verify installation:**
```cmd
java -version
```

#### macOS

```bash
# Using Homebrew
brew install openjdk@21

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 21)' >> ~/.zshrc
source ~/.zshrc

# Verify
java -version
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt install openjdk-21-jdk

# Fedora
sudo dnf install java-21-openjdk-devel

# Verify
java -version
```

### Install Gradle

#### Windows

**Option 1: Package Manager (recommended)**

```powershell
# Chocolatey
choco install gradle

# Scoop
scoop install gradle

# winget
winget install Gradle.Gradle
```

**Option 2: Manual Install**

1. Download from [gradle.org/releases](https://gradle.org/releases/)
2. Extract to `C:\Gradle\gradle-9.x`
3. Add to PATH:
   ```cmd
   setx PATH "%PATH%;C:\Gradle\gradle-9.x\bin"
   ```

**Verify installation:**
```cmd
gradle -version
```

#### macOS

```bash
brew install gradle
gradle -version
```

#### Linux

```bash
# Using SDKMAN (recommended)
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install gradle 9.0

# Verify
gradle -version
```

> **Tip:** After setting up the Gradle wrapper in your project, you won't need Gradle installed globally - use `./gradlew` or `gradlew.bat` instead.

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

```powershell
$env:JAVA_HOME = "C:\Program Files\Microsoft\jdk-21.0.x-hotspot"
.\gradlew.bat build -x test
```

### Linux/Mac

```bash
JAVA_HOME=/path/to/jdk21 ./gradlew build -x test
```

### Deploy

Copy the JAR from `build/libs/` to your server's Mods folder:

**Windows:**
```
%APPDATA%\Hytale\Userdata\Saves\<world_name>\Mods\
```

**Linux/Mac:**
```
~/.hytale/Userdata/Saves/<world_name>/Mods/
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
