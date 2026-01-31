# NPC Spawn Markers

Spawn markers are world-placed assets that control where and how NPCs spawn in the game world. They are typically placed in prefabs, structures, or directly during world generation.

## Overview

Spawn markers define:
- **Which NPCs** can spawn at a location
- **When** they spawn (game time vs real time)
- **How often** they respawn
- **When they despawn** (distance-based)

## Asset Structure

Spawn markers are located at `NPC/Spawn/Markers/` in the asset hierarchy.

### Basic Example

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    {
      "Name": "Bear_Grizzly",
      "Weight": 100,
      "RealtimeRespawnTime": 420
    }
  ],
  "ExclusionRadius": 20,
  "RealtimeRespawn": true,
  "MaxDropHeight": 4
}
```

### Persistent NPC Example (Merchants)

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    {
      "Name": "Temple_Kweebec_Merchant",
      "Weight": 100,
      "SpawnAfterGameTime": "P1D"
    }
  ],
  "ExclusionRadius": 10,
  "MaxDropHeight": 4,
  "DeactivationDistance": 1000
}
```

### Multiple NPC Variants

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    {
      "Name": "Goblin_Scrapper",
      "Weight": 60,
      "SpawnAfterGameTime": "P1D"
    },
    {
      "Name": "Goblin_Lobber",
      "Weight": 25,
      "SpawnAfterGameTime": "P1D"
    },
    {
      "Name": "Goblin_Thief",
      "Weight": 15,
      "SpawnAfterGameTime": "P1D"
    }
  ],
  "ExclusionRadius": 10,
  "MaxDropHeight": 4
}
```

## Field Reference

### Marker-Level Fields

| Field | Type | Description |
|-------|------|-------------|
| `Model` | string | Always `"NPC_Spawn_Marker"` |
| `NPCs` | array | List of possible NPCs to spawn |
| `ExclusionRadius` | number | Minimum distance from other spawn markers (blocks) |
| `MaxDropHeight` | number | Maximum fall distance when finding spawn position |
| `RealtimeRespawn` | boolean | If true, uses real seconds; if false, uses game time |
| `DeactivationDistance` | number | Distance from player before NPC despawns. Default ~128. Set high (1000+) for persistent NPCs |

### NPC Entry Fields

| Field | Type | Description |
|-------|------|-------------|
| `Name` | string | NPC role name to spawn |
| `Weight` | number | Spawn probability weight (higher = more likely) |
| `RealtimeRespawnTime` | number | Seconds (real time) before respawn after death/despawn |
| `SpawnAfterGameTime` | string | ISO 8601 duration before first spawn (e.g., `"P1D"` = 1 game day) |

### Time Format (ISO 8601 Duration)

- `P1D` - 1 day
- `PT1H` - 1 hour
- `PT30M` - 30 minutes
- `P1DT12H` - 1 day and 12 hours

## DeactivationDistance Deep Dive

The `DeactivationDistance` field is key for controlling NPC persistence:

| Value | Behavior |
|-------|----------|
| Not set (~128) | Default behavior - despawns when player is ~128 blocks away |
| 500-1000 | Semi-persistent - stays loaded in a larger area |
| 1000+ | Effectively persistent within normal play |

**Important:** This only works for NPCs spawned BY the marker. Dynamically created NPCs (via Java) don't have a spawn marker and use different despawn logic.

## Use Cases

### 1. Wildlife Spawns

Standard animal spawns with respawn timers:

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    { "Name": "Deer_Doe", "Weight": 70, "RealtimeRespawnTime": 300 },
    { "Name": "Deer_Stag", "Weight": 30, "RealtimeRespawnTime": 300 }
  ],
  "ExclusionRadius": 15,
  "RealtimeRespawn": true,
  "MaxDropHeight": 4
}
```

### 2. Dungeon Enemies

Enemies that respawn after game time:

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    { "Name": "Skeleton_Soldier", "Weight": 100, "SpawnAfterGameTime": "P1D" }
  ],
  "ExclusionRadius": 8,
  "MaxDropHeight": 2
}
```

### 3. Persistent Merchants/Quest NPCs

NPCs that should never despawn:

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    { "Name": "Village_Blacksmith", "Weight": 100, "SpawnAfterGameTime": "P1D" }
  ],
  "ExclusionRadius": 5,
  "MaxDropHeight": 2,
  "DeactivationDistance": 2000
}
```

### 4. Farm Prefab with Pre-Tamed Animals

For structures that come with tamed livestock:

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    { "Name": "Cow_Tamed", "Weight": 100 }
  ],
  "ExclusionRadius": 5,
  "MaxDropHeight": 2,
  "DeactivationDistance": 1500
}
```

## Limitations

### Cannot Be Used For:

1. **Dynamically tamed animals** - Spawn markers are static world objects placed at fixed locations. When a player tames a wild animal, there's no spawn marker associated with it.

2. **Mobile NPCs** - The marker is location-bound, not entity-bound. An NPC that follows the player can't benefit from marker-based persistence.

3. **Runtime creation** - Spawn markers are loaded from assets at world generation/chunk load time. You cannot create them programmatically at runtime.

### For Dynamic NPCs (Java-Side Persistence)

For dynamically created or modified NPCs (like tamed animals), use Java:

```java
// Disable spawn tracking to prevent despawn
npcEntity.updateSpawnTrackingState(false);
```

This tells the spawn system not to track/despawn the NPC, but it has edge cases and isn't as robust as marker-based persistence.

## File Locations

Spawn markers are organized by type:

```
NPC/Spawn/Markers/
├── Aquatic/           # Fish, water creatures
├── Bat                # Cave bats
├── Bear               # Bears
├── Birds_Cave         # Cave birds
├── Elemental/         # Golems
├── Instance/          # Instance-specific (dungeons, temples)
│   ├── Dungeon_Goblin/
│   └── Forgotten_Temple/
└── Intelligent/       # Hostile humanoids
    ├── Feran/
    ├── Goblin/
    ├── Outlander/
    └── Trork/
```

## Creating Custom Spawn Markers

To add spawn markers for modded content:

1. Create JSON file in your mod's `Server/NPC/Spawn/Markers/` directory
2. Reference existing NPC roles or your custom roles
3. Place the marker in prefabs or use world generation hooks

Example for a custom tamed animal farm:

```json
// Server/NPC/Spawn/Markers/Custom/Farm_Livestock.json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [
    { "Name": "Cow_Tamed", "Weight": 40 },
    { "Name": "Pig_Tamed", "Weight": 30 },
    { "Name": "Chicken_Tamed", "Weight": 30 }
  ],
  "ExclusionRadius": 8,
  "MaxDropHeight": 2,
  "DeactivationDistance": 1500
}
```

## See Also

- [NPC Attitude System](npc-attitude-system.md)
- [Asset-Based Taming](asset-based-taming.md)
- [ECS Patterns](ecs-patterns.md) - For Java-side NPC manipulation
