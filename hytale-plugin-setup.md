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

**Verify:** Open a new terminal and run:
```cmd
java -version
```
Expected output (version should be 21.x.x):
```
openjdk version "21.0.x" 2024-xx-xx
```

#### macOS

```bash
# Using Homebrew
brew install openjdk@21

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 21)' >> ~/.zshrc
source ~/.zshrc
```

**Verify:**
```bash
java -version
```
Expected: `openjdk version "21.x.x"`

#### Linux

```bash
# Ubuntu/Debian
sudo apt install openjdk-21-jdk

# Fedora
sudo dnf install java-21-openjdk-devel
```

**Verify:**
```bash
java -version
```
Expected: `openjdk version "21.x.x"`

---

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

**Verify:** Open a new terminal and run:
```cmd
gradle -version
```
Expected output (version should be 9.x):
```
------------------------------------------------------------
Gradle 9.x
------------------------------------------------------------
```

#### macOS

```bash
brew install gradle
```

**Verify:**
```bash
gradle -version
```
Expected: `Gradle 9.x`

#### Linux

```bash
# Using SDKMAN (recommended)
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install gradle 9.0
```

**Verify:**
```bash
gradle -version
```
Expected: `Gradle 9.x`

> **Tip:** After setting up the Gradle wrapper in your project, you won't need Gradle installed globally - use `./gradlew` or `gradlew.bat` instead.

---

## Quick Start

### 1. Create Project Structure

Create the following folder structure:

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

**Verify (Windows):**
```powershell
tree /F
```

**Verify (Linux/Mac):**
```bash
find . -type f | head -20
```
Expected: You should see all the files listed above.

---

### 2. Initialize Gradle Wrapper

Run this once in your project folder:

```bash
gradle wrapper --gradle-version 9.0
```

**Verify:** Check that these files were created:
- `gradlew` (Linux/Mac)
- `gradlew.bat` (Windows)
- `gradle/wrapper/gradle-wrapper.jar`
- `gradle/wrapper/gradle-wrapper.properties`

```bash
# Linux/Mac
ls gradlew gradle/wrapper/

# Windows
dir gradlew.bat gradle\wrapper\
```

---

### 3. settings.gradle

```groovy
rootProject.name = 'my-hytale-plugin'
```

**Verify:**
```bash
cat settings.gradle
```
Expected: Should show `rootProject.name = 'my-hytale-plugin'`

---

### 4. gradle.properties

```properties
org.gradle.jvmargs=-Xmx2G
org.gradle.parallel=true
org.gradle.caching=true
```

**Verify:**
```bash
cat gradle.properties
```
Expected: Should show the three properties above.

---

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

**Verify:** Test that Gradle can parse the build file:
```bash
# Windows
.\gradlew.bat tasks --quiet

# Linux/Mac
./gradlew tasks --quiet
```
Expected: Should list available tasks without errors. If you see errors about `HytaleServer.jar`, that's OK at this stage - it means Gradle is working but the JAR is missing.

---

### 6. manifest.json

Place in `src/main/resources/manifest.json`:

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

**Verify:**
```bash
cat src/main/resources/manifest.json
```
Expected: Should show the JSON above with `"Main"` matching your package/class path.

---

### 7. Main Plugin Class

Create `src/main/java/com/yourname/pluginname/MyPlugin.java`:

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

**Verify:** Ensure the package declaration matches the file path:
- File: `src/main/java/com/yourname/pluginname/MyPlugin.java`
- Package: `package com.yourname.pluginname;`
- Manifest Main: `"Main": "com.yourname.pluginname.MyPlugin"`

---

## Build & Deploy

### Build

**Windows:**
```powershell
$env:JAVA_HOME = "C:\Program Files\Microsoft\jdk-21.0.x-hotspot"
.\gradlew.bat build -x test
```

**Linux/Mac:**
```bash
JAVA_HOME=/path/to/jdk21 ./gradlew build -x test
```

**Verify:** Check that the JAR was created:
```bash
# Windows
dir build\libs\

# Linux/Mac
ls -la build/libs/
```
Expected: Should show `my-hytale-plugin-1.0.0.jar` (or similar).

---

### Deploy

Copy the JAR from `build/libs/` to the `Mods` folder inside your world's save directory.

**Windows:**
```
%APPDATA%\Hytale\UserData\Saves\<world_name>\Mods\
```

**Verify:** Start the Hytale server and check the console for:
```
[INFO] Plugin setup complete!
[INFO] Plugin started!
```

If you see these messages, your plugin loaded successfully!

---

### Deploy Script (Windows)

For faster iteration, create a `deploy.ps1` script in your project root. This script uses environment variables so you only configure once:

```powershell
# deploy.ps1 - Build and deploy script
# Usage: .\deploy.ps1
# Interactive mode: Press SPACE to deploy, Q to quit

$ErrorActionPreference = "Stop"

# ============================================
# CONFIGURATION
# Uses environment variables or prompts user
# ============================================

function Get-ConfigValue {
    param(
        [string]$EnvVarName,
        [string]$PromptMessage,
        [string]$DefaultValue = ""
    )

    $value = [Environment]::GetEnvironmentVariable($EnvVarName, "User")

    if ([string]::IsNullOrWhiteSpace($value)) {
        Write-Host ""
        Write-Host "Environment variable '$EnvVarName' is not set." -ForegroundColor Yellow

        if ($DefaultValue) {
            Write-Host "Default: $DefaultValue" -ForegroundColor DarkGray
        }

        $input = Read-Host $PromptMessage

        if ([string]::IsNullOrWhiteSpace($input) -and $DefaultValue) {
            $value = $DefaultValue
        } else {
            $value = $input
        }

        if ([string]::IsNullOrWhiteSpace($value)) {
            Write-Host "Value cannot be empty. Exiting." -ForegroundColor Red
            exit 1
        }

        $save = Read-Host "Save '$value' to environment variable '$EnvVarName'? (Y/n)"
        if ($save -ne 'n' -and $save -ne 'N') {
            [Environment]::SetEnvironmentVariable($EnvVarName, $value, "User")
            Write-Host "Saved to user environment variables." -ForegroundColor Green
        }
    }

    return $value
}

Write-Host ""
Write-Host "============================================" -ForegroundColor Cyan
Write-Host "       HYTALE PLUGIN DEPLOY SETUP          " -ForegroundColor Cyan
Write-Host "============================================" -ForegroundColor Cyan

# Use existing JAVA_HOME as default if available
$javaDefault = if ($env:JAVA_HOME) { $env:JAVA_HOME } else { "C:\Program Files\Microsoft\jdk-21.0.9.10-hotspot" }
$hytaleInstallDefault = "C:\Users\$env:USERNAME\AppData\Roaming\Hytale";

$JAVA_HOME = Get-ConfigValue -EnvVarName "HYTALE_JAVA_HOME" `
    -PromptMessage "Enter Java 21 home path" `
    -DefaultValue $javaDefault

$PLUGIN_NAME = Get-ConfigValue -EnvVarName "HYTALE_PLUGIN_NAME" `
    -PromptMessage "Enter plugin name (jar filename without version)" `
    -DefaultValue "my-mod"

$HYTALE_INSTALL_PATH = Get-ConfigValue -EnvVarName "HYTALE_INSTALL_PATH" `
    -PromptMessage "Enter Hytale install path" `
    -DefaultValue $hytaleInstallDefault

# Optional server path
$SERVER_PATH = [Environment]::GetEnvironmentVariable("HYTALE_SERVER_PATH", "User")

if ([string]::IsNullOrWhiteSpace($SERVER_PATH)) {
    Write-Host ""
    $addServer = Read-Host "Do you want to add a server mods path? (y/N)"
    if ($addServer -eq 'y' -or $addServer -eq 'Y') {
        $SERVER_PATH = Get-ConfigValue -EnvVarName "HYTALE_SERVER_PATH" `
            -PromptMessage "Enter server mods path (e.g., C:\HytaleServer\mods)"
    }
}

# ============================================

$env:JAVA_HOME = $JAVA_HOME
$dest = "$HYTALE_INSTALL_PATH\UserData\Mods"
$serverDest = $SERVER_PATH

function Deploy {
    Clear-Host
    Write-Host ""
    Write-Host "============================================" -ForegroundColor Cyan
    Write-Host "            DEPLOYING PLUGIN                " -ForegroundColor Cyan
    Write-Host "============================================" -ForegroundColor Cyan
    Write-Host ""

    # Read version from build.gradle
    $buildGradle = Get-Content "build.gradle" -Raw
    if ($buildGradle -match "version\s*=\s*'([^']+)'") {
        $version = $matches[1]
    } else {
        Write-Host "Could not read version from build.gradle" -ForegroundColor Red
        return $false
    }

    Write-Host "Plugin: $PLUGIN_NAME" -ForegroundColor Yellow
    Write-Host "Version: $version" -ForegroundColor Yellow
    Write-Host ""

    # Build
    Write-Host "Building..." -ForegroundColor Yellow
    & .\gradlew.bat build -x test --no-daemon -q

    if ($LASTEXITCODE -ne 0) {
        Write-Host ""
        Write-Host "BUILD FAILED!" -ForegroundColor Red
        return $false
    }

    Write-Host "Build successful!" -ForegroundColor Green

    # Create mods folder if needed
    if (-not (Test-Path $dest)) {
        New-Item -ItemType Directory -Path $dest -Force | Out-Null
        Write-Host "Created Mods folder" -ForegroundColor Yellow
    }

    # Delete old versions
    $oldJars = Get-ChildItem -Path $dest -Filter "$PLUGIN_NAME*.jar" -ErrorAction SilentlyContinue
    foreach ($jar in $oldJars) {
        try {
            Remove-Item $jar.FullName -Force -ErrorAction Stop
            Write-Host "Deleted old: $($jar.Name)" -ForegroundColor DarkYellow
        } catch {
            Write-Host "Could not delete $($jar.Name) (in use)" -ForegroundColor DarkYellow
        }
    }

    # Copy new JAR to client
    $source = "build\libs\$PLUGIN_NAME-$version.jar"
    $destFile = Join-Path $dest "$PLUGIN_NAME-$version.jar"
    Copy-Item $source $destFile -Force

    Write-Host ""
    Write-Host "DEPLOYED (client): $destFile" -ForegroundColor Green

    # Copy to server if configured
    if (-not [string]::IsNullOrWhiteSpace($serverDest)) {
        if (-not (Test-Path $serverDest)) {
            New-Item -ItemType Directory -Path $serverDest -Force | Out-Null
            Write-Host "Created server Mods folder" -ForegroundColor Yellow
        }

        # Delete old versions from server
        $oldServerJars = Get-ChildItem -Path $serverDest -Filter "$PLUGIN_NAME*.jar" -ErrorAction SilentlyContinue
        foreach ($jar in $oldServerJars) {
            try {
                Remove-Item $jar.FullName -Force -ErrorAction Stop
                Write-Host "Deleted old (server): $($jar.Name)" -ForegroundColor DarkYellow
            } catch {
                Write-Host "Could not delete $($jar.Name) from server (in use)" -ForegroundColor DarkYellow
            }
        }

        $serverDestFile = Join-Path $serverDest "$PLUGIN_NAME-$version.jar"
        Copy-Item $source $serverDestFile -Force
        Write-Host "DEPLOYED (server): $serverDestFile" -ForegroundColor Green
    }
    Write-Host ""
    Write-Host "Reload commands:" -ForegroundColor Yellow
    Write-Host "  /plugin unload YourOrg:MyPlugin" -ForegroundColor DarkGray
    Write-Host "  /plugin load YourOrg:MyPlugin" -ForegroundColor DarkGray
    Write-Host ""

    return $true
}

function ShowPrompt {
    Write-Host "============================================" -ForegroundColor DarkCyan
    Write-Host "  Press [SPACE] to deploy  |  [Q] to quit  " -ForegroundColor White
    Write-Host "============================================" -ForegroundColor DarkCyan
}

# Main
Clear-Host
Write-Host ""
Write-Host "  HYTALE PLUGIN DEPLOY WATCHER" -ForegroundColor Cyan
Write-Host ""
ShowPrompt

while ($true) {
    if ([Console]::KeyAvailable) {
        $key = [Console]::ReadKey($true)

        if ($key.Key -eq [ConsoleKey]::Spacebar) {
            $success = Deploy
            if ($success) {
                Write-Host "Deployed at $(Get-Date -Format 'HH:mm:ss')" -ForegroundColor Green
            } else {
                Write-Host "Failed at $(Get-Date -Format 'HH:mm:ss')" -ForegroundColor Red
            }
            Write-Host ""
            ShowPrompt
        }
        elseif ($key.Key -eq [ConsoleKey]::Q) {
            Write-Host "Exiting..." -ForegroundColor Yellow
            break
        }
    }
    Start-Sleep -Milliseconds 100
}
```

**First run:** The script will prompt for configuration values and offer to save them as user environment variables:
- `HYTALE_JAVA_HOME` - Path to your JDK 21
- `HYTALE_PLUGIN_NAME` - Your JAR name (without version/extension)
- `HYTALE_INSTALL_PATH` - Hytale install path
- `HYTALE_SERVER_PATH` - (Optional) Server mods path for dual deployment

**Usage:**
```powershell
.\deploy.ps1
```

Then press **SPACE** to build and deploy, **Q** to quit.

**Features:**
- Auto-saves configuration to user environment variables
- Deploys to both client and server (if configured)
- Cleans old JAR versions before deploying

**Verify:** After deploying, reload in-game:
```
/plugin unload YourOrg:MyPlugin
/plugin load YourOrg:MyPlugin
```

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

**Verify:** Connect to the server and check the console for `Player connected: <your_name>`.

---

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

**Verify:** In-game, run `/hello`. Expected response: `Hello, World!`

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

**Verify:** After building, check your JAR contains the assets:
```bash
# Windows (requires 7-Zip or similar)
jar tf build/libs/my-hytale-plugin-1.0.0.jar | findstr "Common Server"

# Linux/Mac
jar tf build/libs/my-hytale-plugin-1.0.0.jar | grep -E "Common|Server"
```
Expected: Should list your asset files.

See [How To: Add Custom Sounds](api/api-empirical-how-tos.md#how-to-add-custom-sounds) for details.

---

## Logging

```java
getLogger().atInfo().log("Info message");
getLogger().atInfo().log("Formatted: %s", value);
getLogger().atWarning().log("Warning message");
getLogger().atSevere().log("Error message");
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `java -version` shows wrong version | Ensure `JAVA_HOME` is set correctly and restart terminal |
| `gradle` command not found | Ensure Gradle is in PATH and restart terminal |
| Build fails with "cannot find symbol" | Check that `HytaleServer.jar` is in `libs/` folder |
| Plugin doesn't load | Check that `Main` in manifest.json matches your class path exactly |
| `${version}` not replaced | Ensure `processResources` block is in build.gradle |

---

## Next Steps

- [Plugin System](api/01-plugin-system.md) - Detailed plugin lifecycle
- [Commands](api/04-commands.md) - Advanced command patterns
- [Events](api/05-events.md) - Event handling system
- [ECS System](api/02-ecs-system.md) - Entity Component System
- [How-To Examples](api/api-empirical-how-tos.md) - Practical code examples
