# Theoretical / Untested / Experimental

> **Warning:** This section documents systems that have been identified through reverse engineering but have NOT been fully tested in plugin development. The information here is based on static code analysis and may be incomplete or inaccurate.

## Overview

The following systems have been identified in the HytaleServer.jar codebase. Some have partial documentation elsewhere - check the cross-references!

---

## Verified Systems (Moved from Experimental)

These systems were initially experimental but have been tested and verified. See the linked documentation for working code.

### Audio / Sound System - VERIFIED

> **Status:** Working! See [API Discoveries - Sound System](99-api-discoveries.md#sound-system-verified)

**Working Pattern:**
```java
import com.hypixel.hytale.server.core.asset.type.soundevent.config.SoundEvent;
import com.hypixel.hytale.server.core.universe.world.SoundUtil;
import com.hypixel.hytale.protocol.SoundCategory;

int soundId = SoundEvent.getAssetMap().getIndex("SFX_Consume_Bread");
SoundUtil.playSoundEvent3d(soundId, SoundCategory.SFX, x, y, z, store);
```

**Still Untested:**
- Custom sound registration
- Looping sounds and music
- Sound category volume control

---

### Particle Effects System - VERIFIED

> **Status:** Working! See [API Discoveries - Particle System](99-api-discoveries.md#particle-system-verified)

**Working Pattern:**
```java
import com.hypixel.hytale.server.core.universe.world.ParticleUtil;

ParticleUtil.spawnParticleEffect("MyParticleSystem", position, store);
```

**Asset Files:**
- `Server/Particles/{name}.particlesystem` - Particle system definition
- `Server/Particles/Spawners/{name}.particlespawner` - Spawner configuration

**Still Untested:**
- Custom particle parameters (color, size at runtime)
- Particle attachment to moving entities

---

## Partially Documented Systems

### Entity Spawn Detection - VERIFIED

> **Status:** Working! See [API Discoveries - Entity Spawn Detection](99-api-discoveries.md#entity-spawn-detection-verified)

Use `EntityTickingSystem` with `NewSpawnComponent` query. See also [ECS System](02-ecs-system.md#entitytickingsystem-verified).

---

### Animation State Machines - PARTIAL

**Documented in:** [Entities - Entity Systems](03-entities.md#entity-systems-location)

**What's Known:**
- `ActiveAnimationComponent` tracks current animation
- `ActiveEntityEffect` / `EffectControllerComponent` in entity effect system
- NPCs can trigger animations via `ActionPlayAnimation`
- Damage causes have `animationId` and `deathAnimationId` fields

**What Needs Testing:**
- Triggering animations from plugins directly
- Animation blending/layering
- Custom animation registration
- Animation events/callbacks

**Related Classes:**
```
com.hypixel.hytale.server.core.modules.entity.component.ActiveAnimationComponent
com.hypixel.hytale.server.core.entity.AnimationUtils
com.hypixel.hytale.server.npc.animations.NPCAnimationSlot
```

---

### Status Effects / Buffs - PARTIAL

**Documented in:** [Entities - Entity Systems](03-entities.md#entity-systems-location)

**What's Known:**
- `ActiveEntityEffect` - Active effects on entities
- `EffectControllerComponent` - Effect management
- `LivingEntityEffectSystem` - Effect handling system
- `ActionApplyEntityEffect` in NPC AI can apply effects
- `EntityStatEffect` interaction type exists

**What Needs Testing:**
- Applying effects from plugins
- Custom effect definitions
- Effect duration and stacking
- Effect removal

**Related Classes:**
```
com.hypixel.hytale.server.core.entity.effect.ActiveEntityEffect
com.hypixel.hytale.server.core.entity.effect.EffectControllerComponent
com.hypixel.hytale.server.core.modules.entity.LivingEntityEffectSystem
```

---

### Scripting / Asset API - PARTIAL

**Documented in:** [Interactions - Asset File Paths](06-interactions.md#asset-file-paths)

**Verified Asset Paths:**
| Asset Type | Path |
|------------|------|
| RootInteraction | `Server/Item/RootInteractions/{id}.json` |
| Interaction | `Server/Item/Interactions/{id}.json` |
| NPC Role | `Server/NPC/Roles/{id}.json` |
| Particles | `Server/Particles/{name}.particlesystem` |

**What's Known:**
- Most game behavior is defined via JSON assets
- `AssetMap` used for asset ID lookups
- `CodecMapRegistry` for registering custom types
- Various `*JsonLoader` classes handle loading

**What Needs Testing:**
- Runtime asset modification
- Custom asset type registration
- Asset hot-reloading

---

### ECS Component System - DOCUMENTED

> **Status:** Fully documented! See:
> - [ECS System](02-ecs-system.md) - Core patterns
> - [API Discoveries - ECS Component System](99-api-discoveries.md#ecs-component-system)
> - [Reverse Engineering - Codebase Analysis](../reverse-engineering/codebase-analysis.md#entity-component-system)

The ECS system is well-documented with working examples.

---

## Systems Requiring Further Analysis

### Chunk Storage / Persistence

**Partially in:** [API Discoveries - World/Universe](99-api-discoveries.md#worlduniverse)

**Identified Packages:**
- `com.hypixel.hytale.server.core.universe.world.storage.ChunkStore`
- `com.hypixel.hytale.server.core.universe.world.chunk.WorldChunk`
- `GeneratedChunk` with block/entity data

**What We Know:**
- `EntityStore` wraps inner `Store` (see API Discoveries)
- Chunks stored via `ChunkStore` with ECS pattern
- `WorldChunk` contains block and entity data
- Chunk sections use `Holder<ChunkStore>[]`

**What Needs Testing:**
- Custom chunk data persistence
- Block state storage format
- Entity persistence in chunks
- Chunk loading/unloading hooks

---

### Multiplayer Synchronization

**Partially in:** [API Discoveries - ECS Component System](99-api-discoveries.md#ecs-component-system)

**What We Know:**
- `SnapshotBuffer` component (index 34) for network state tracking
- `NetworkId` component (index 17) for entity identification
- Packets organized by category (player, entity, inventory, blocks)
- `ModelTransform` tracks sent transform state
- `isNetworkOutdated` field on components

**What Needs Testing:**
- Custom packet registration
- State synchronization patterns
- Client prediction handling
- Bandwidth optimization

**Related:** See [Reverse Engineering - Network Protocol](../reverse-engineering/codebase-analysis.md#network-protocol)

---

### Weather System

**Potential Location:**
- `EnvironmentContainer` in biome system
- Weather-related packets (if any)

**Status:** Not yet investigated. Biomes have `EnvironmentContainer` which may control weather.

**Related:** See [Reverse Engineering - World Generation](../reverse-engineering/codebase-analysis.md#world-generation-system)

---

### Lighting System

**Identified Components:**
- `DynamicLight` component
- `PersistentDynamicLight` component

**What We Know:**
- Entities can have dynamic lights attached
- Lights can be persistent or temporary
- Components exist at known indices in ECS

**What Needs Testing:**
- Creating lights from plugins
- Light color and intensity control
- Light attachment to entities
- Performance implications

**Related:** See [Reverse Engineering - Entity Components](../reverse-engineering/codebase-analysis.md#entity-components)

---

### Quest / Objective System

**Status:** Not yet investigated. May exist in `builtin/` plugins or not exposed to server API.

---

## Contributing

If you've tested any of these systems and have working code examples, please contribute!

1. Test the system in an actual plugin
2. Document your findings with code examples
3. Submit a PR moving the section to the appropriate tested documentation

See our [Contributing Guide](../CONTRIBUTING.md) for details.

---

## Cross-Reference Index

| System | Status | Main Documentation |
|--------|--------|-------------------|
| Sound/Audio | **VERIFIED** | [API Discoveries](99-api-discoveries.md#sound-system-verified) |
| Particles | **VERIFIED** | [API Discoveries](99-api-discoveries.md#particle-system-verified) |
| Spawn Detection | **VERIFIED** | [API Discoveries](99-api-discoveries.md#entity-spawn-detection-verified) |
| ECS Components | **DOCUMENTED** | [ECS System](02-ecs-system.md), [Reverse Eng.](../reverse-engineering/codebase-analysis.md) |
| Interactions | **DOCUMENTED** | [Interactions](06-interactions.md) |
| Animations | Partial | [Entities](03-entities.md#entity-systems-location) |
| Status Effects | Partial | [Entities](03-entities.md#entity-systems-location) |
| Asset System | Partial | [Interactions](06-interactions.md#asset-file-paths) |
| Chunk Storage | Partial | [API Discoveries](99-api-discoveries.md#worlduniverse) |
| Network Sync | Partial | [Reverse Eng.](../reverse-engineering/codebase-analysis.md#network-protocol) |
| Weather | Untested | [Reverse Eng.](../reverse-engineering/codebase-analysis.md#world-generation-system) |
| Lighting | Untested | [Reverse Eng.](../reverse-engineering/codebase-analysis.md#entity-components) |
| Quests | Unknown | - |

---

## Reverse Engineering Resources

For deeper analysis of any system, see:
- [Codebase Analysis](../reverse-engineering/codebase-analysis.md) - Full system documentation
- [Quick Lookup](../reverse-engineering/quick-lookup.md) - Fast reference by use case
- [Common Questions](../reverse-engineering/answers.md) - Specific answers
