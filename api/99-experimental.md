# Theoretical / Untested / Experimental

> **Warning:** This section documents systems that have been identified through reverse engineering but have NOT been tested in actual plugin development. The information here is based on static code analysis and may be incomplete or inaccurate.

## Overview

The following systems have been identified in the HytaleServer.jar codebase but require further exploration and testing before they can be used reliably in plugins.

---

## Systems Requiring Further Analysis

### Audio / Sound System

**Identified Packages:**
- `com.hypixel.hytale.server.core.modules.entity.component.AudioComponent`
- `com.hypixel.hytale.server.core.modules.entity.component.MovementAudioComponent`
- Sound event system via `SoundUtil.playSoundEvent3d()`

**What We Know:**
- Entities have `AudioComponent` for sound emission
- `MovementAudioComponent` handles footstep sounds
- Sounds are played via `SoundEvent.getAssetMap().getIndex("SoundId")`

**What Needs Testing:**
- Custom sound registration
- 3D positional audio parameters
- Sound categories and volume control
- Looping sounds and music

---

### Particle Effects System

**Identified Patterns:**
- Particles referenced in `Damage.IMPACT_PARTICLES`
- `ActionSpawnParticles` in NPC AI system
- Particle metadata in damage events

**What Needs Testing:**
- How to spawn particles from plugins
- Available particle types
- Custom particle parameters (color, size, lifetime)
- Particle attachment to entities

---

### Animation State Machines

**Identified Packages:**
- `com.hypixel.hytale.server.core.modules.entity.component.ActiveAnimationComponent`
- `ActionPlayAnimation` in NPC system
- Animation IDs in damage causes

**What We Know:**
- Entities have `ActiveAnimationComponent` tracking current animation
- NPCs can trigger animations via `ActionPlayAnimation`
- Damage causes have `animationId` and `deathAnimationId` fields

**What Needs Testing:**
- Triggering animations from plugins
- Animation blending/layering
- Custom animation registration
- Animation events/callbacks

---

### Scripting / Asset API

**Identified Patterns:**
- Asset-driven configuration throughout the codebase
- JSON loaders in `server.worldgen.loader/`
- `AssetMap` used for asset ID lookups

**What We Know:**
- Most game behavior is defined via JSON assets
- Asset paths follow patterns like `Server/Item/Types/{id}.json`
- Assets are loaded via various `*JsonLoader` classes

**What Needs Testing:**
- Runtime asset modification
- Custom asset registration
- Asset hot-reloading
- Scripting hooks if any exist

---

### Chunk Storage / Persistence

**Identified Packages:**
- `com.hypixel.hytale.server.core.universe.world.storage.ChunkStore`
- `com.hypixel.hytale.server.core.universe.world.chunk.WorldChunk`
- `GeneratedChunk` with block/entity data

**What We Know:**
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

**Identified Packages:**
- `com.hypixel.hytale.protocol.packets/`
- `SnapshotBuffer` component for network state
- Various `Set*` packets for state sync

**What We Know:**
- Entities have `SnapshotBuffer` for network state tracking
- Packets organized by category (player, entity, inventory, blocks)
- `ModelTransform` tracks sent transform state

**What Needs Testing:**
- Custom packet registration
- State synchronization patterns
- Client prediction handling
- Bandwidth optimization

---

### Weather System

**Potential Location:**
- `EnvironmentContainer` in biome system
- Weather-related packets (if any)

**Status:** Not yet investigated. Biomes have `EnvironmentContainer` which may control weather.

---

### Lighting System

**Identified Components:**
- `DynamicLight` component
- `PersistentDynamicLight` component

**What We Know:**
- Entities can have dynamic lights attached
- Lights can be persistent or temporary

**What Needs Testing:**
- Creating lights from plugins
- Light color and intensity control
- Light attachment to entities
- Performance implications

---

### Status Effects / Buffs

**Potential Locations:**
- `ActionApplyEntityEffect` in NPC AI
- `EntityStatEffect` interaction type
- Stat-based entity filters

**Status:** Partially identified. NPCs can apply effects, but the full effect system needs mapping.

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

## Reverse Engineering Notes

These systems were identified through analysis of HytaleServer.jar (~37,900 classes). Key techniques used:

- `javap` for class structure analysis
- Pattern matching on class/method names
- Cross-referencing with known working code
- Asset file analysis

For detailed reverse engineering documentation, see the `reverse-engineer/docs/` folder:
- `CODEBASE_ANALYSIS.md` - Full system documentation
- `CODEBASE_ANALYSIS_MAP.md` - Quick lookup index
- `ANSWERS.md` - Specific question answers
