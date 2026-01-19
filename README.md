# Hytale Modding Documentation

[![View Docs](https://img.shields.io/badge/View%20Docs-GitHub%20Pages-blue)](https://lait-kelomins.github.io/hytale-modding-docs/) [![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-black)](https://github.com/lait-kelomins/hytale-modding-docs)

> Unofficial documentation for the Hytale Server modding API, compiled through experimentation and reverse engineering.

## About

This documentation covers the Hytale Server Plugin API discovered through community modding efforts. The API is not officially documented, so this serves as a community resource.

**Pragmatic approach:** This documentation focuses on patterns that *work today* with the current game code. We prioritize getting things done over finding the "perfect" solution. There are likely cleaner or more efficient approaches for many of the techniques documented here - we just haven't found them yet.

**Help us improve:** If you discover better patterns, find errors, or want to contribute new documentation, we'd love to hear from you! Open an [issue](https://github.com/lait-kelomins/hytale-modding-docs/issues) or submit a pull request. See our [Contributing Guide](CONTRIBUTING.md) for details.

## Quick Navigation

| I want to... | Read this |
|--------------|-----------|
| Create my first plugin | [Quick Start Guide](hytale-plugin-setup.md) |
| Understand the plugin lifecycle | [Plugin System](api/01-plugin-system.md) |
| Handle events (blocks, players) | [Events](api/05-events.md) |
| Create commands | [Commands](api/04-commands.md) |
| Work with entities | [Entities](api/03-entities.md) |
| Understand the ECS architecture | [ECS System](api/02-ecs-system.md) |
| Work with items/inventory | [Items & Inventory](api/06-items.md) |
| Handle interactions | [Interactions](api/06-interactions.md) |

## Core Package Structure

```
com.hypixel.hytale.
├── component/                    # ECS Core System
│   ├── system/
│   │   ├── EntityEventSystem     # Handle entity events
│   │   ├── WorldEventSystem      # Handle world events
│   │   └── TickingSystem         # Periodic updates
│   ├── ArchetypeChunk            # Entity data container
│   ├── CommandBuffer             # Deferred entity commands
│   ├── Store                     # World/entity store
│   └── Query                     # Component queries
│
├── server/core/
│   ├── plugin/
│   │   ├── JavaPlugin            # BASE CLASS for plugins
│   │   ├── PluginManager         # Plugin lifecycle
│   │   └── PluginState           # Plugin states enum
│   │
│   ├── event/events/
│   │   ├── ecs/                  # ECS Events (block, craft, etc)
│   │   ├── player/               # Player Events
│   │   └── entity/               # Entity Events
│   │
│   └── universe/
│       └── PlayerRef             # Player reference component
│
└── common/plugin/
    └── PluginManifest            # Plugin metadata
```

## Requirements

- **Java 21** - HytaleServer.jar is compiled with Java 21
- **Gradle 9.x** - For building plugins
- **HytaleServer.jar** - The server JAR (not redistributable)

## Contributing

Found an error, discovered a better approach, or want to add new documentation? Contributions are welcome!

- **Report issues:** [Open an issue](https://github.com/lait-kelomins/hytale-modding-docs/issues)
- **Contribute:** See our [Contributing Guide](CONTRIBUTING.md)

## Disclaimer

This is unofficial documentation created by the community. Hytale and related trademarks belong to Hypixel Studios.
