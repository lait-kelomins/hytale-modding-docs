# Reverse Engineering Documentation

> Deep analysis of HytaleServer.jar (~37,900 classes) through static code analysis and decompilation.

## Overview

This section contains detailed documentation from reverse engineering the Hytale Server codebase. Unlike the main API documentation which focuses on tested, working patterns, this section provides comprehensive system analysis that may include untested or theoretical information.

## Documents

| Document | Description |
|----------|-------------|
| [Codebase Analysis](codebase-analysis.md) | Full system documentation with code patterns |
| [Quick Lookup](quick-lookup.md) | Fast reference: Use Case → Class Path → Tags |
| [Answers](answers.md) | Specific answers to common modding questions |
| [Status](status.md) | Analysis progress and completed systems |

## What's Covered

### Fully Analyzed Systems

- **ECS Architecture** - Store, Ref, Components, Systems, Queries
- **Player Systems** - Movement, Collision, Interaction
- **NPC AI** - Blackboard pattern, Actions, Sensors
- **Combat & Damage** - Damage pipeline, armor, knockback
- **World Generation** - Zones, Biomes, Climate, Caves, Prefabs
- **Inventory** - Per-world data, containers, transactions
- **Spawn System** - Providers, respawn, world spawn
- **Network Protocol** - Packet categories, state sync
- **Event System** - Player, entity, world events

### Key Architectural Patterns

```
com.hypixel.hytale.
├── component/           # ECS Framework
├── server/core/
│   ├── modules/         # Collision, Physics, Interaction
│   ├── entity/          # Player, LivingEntity
│   ├── event/           # Event system
│   └── universe/        # World management
├── server/npc/          # NPC AI system
├── server/worldgen/     # World generation
├── protocol/            # Network packets
└── builtin/             # Built-in plugins
```

## How to Use

1. **Looking for a specific system?** → Start with [Quick Lookup](quick-lookup.md)
2. **Need detailed patterns?** → Read [Codebase Analysis](codebase-analysis.md)
3. **Have a specific question?** → Check [Answers](answers.md)
4. **Want to see what's documented?** → See [Status](status.md)

## Methodology

Analysis performed using:
- `javap -p` for class/method signatures
- Pattern matching on ~37,900 class names
- Cross-referencing with working plugin code
- Asset file structure analysis

## Disclaimer

This documentation is based on static analysis of decompiled code. Some information may be:
- Incomplete (not all code paths analyzed)
- Inaccurate (decompilation artifacts)
- Outdated (if server updates)

Always test patterns in actual plugins before relying on them.
