# Asset-Based Taming System

> Implementation guide for animal taming using Hytale's native role system
> Last Updated: 2026-01-31

## Overview

This document covers implementing a taming system using Hytale's asset-based approach rather than runtime reflection. When an animal is tamed, it transitions from its wild role (e.g., `Cow`) to a tamed role (e.g., `Cow_Tamed`) via `RoleChangeSystem`.

## Architecture

```
Wild Animal (Cow)                    Tamed Animal (Cow_Tamed)
├─ Extends: Template_Animal_Neutral  ├─ Extends: Template_Animal_Tamed
├─ Attitude: Neutral                 ├─ Attitude: Revered
├─ Hint: "Tame" (when holding food)  ├─ Hint: "Feed" (when holding food)
├─ IsMountable: varies               ├─ IsMountable: true (if applicable)
└─ Feed triggers taming              └─ Feed triggers breeding
```

---

## Key Components

### 1. HyTameComponent (ECS Component)

Tracks per-entity taming state:

```java
public class HyTameComponent implements HyTaleComponent<EntityStore> {
    private boolean tamed = false;
    private UUID tamerUUID = null;
    private String tamerName = null;
    private UUID hytameId = null;      // Unique ID for persistence
    private boolean actionReady = true; // False during cooldowns

    public void setTamed(UUID playerUuid, String playerName) {
        this.tamed = true;
        this.tamerUUID = playerUuid;
        this.tamerName = playerName;
        this.hytameId = UUID.randomUUID();
    }
}
```

### 2. Custom Tamed Sensor

Register a custom sensor to check taming state in asset patches:

**BuilderSensorTamed.java:**
```java
@Register(category = "Sensor", id = "Tamed")
public class BuilderSensorTamed extends BuilderSensor<SensorTamed> {
    @Property(defaultValue = "true")
    private Boolean set;

    @Override
    public SensorTamed build(BuilderSupport support) {
        return new SensorTamed(set);
    }
}
```

**SensorTamed.java:**
```java
public class SensorTamed extends Sensor {
    private final boolean expectedTamed;

    @Override
    public boolean test(Ref<EntityStore> ref, Role role, Store<EntityStore> store) {
        HyTameComponent comp = store.getComponent(ref, HyTameComponent.getComponentType());
        if (comp == null) return !expectedTamed;
        return comp.isTamed() == expectedTamed;
    }
}
```

**Usage in patches:**
```json
{
  "Sensor": { "Type": "Tamed", "Set": true }
}
```

### 3. TamedRoleManager

Manages role transitions:

```java
public class TamedRoleManager {
    // Wild -> Tamed role mapping
    private static final Map<String, String> WILD_TO_TAMED = Map.ofEntries(
        Map.entry("Cow", "Cow_Tamed"),
        Map.entry("Cow_Calf", "Cow_Calf_Tamed"),
        Map.entry("Horse", "Horse_Tamed"),
        // ... all animals
    );

    public boolean applyTamedRole(Ref<EntityStore> npcRef, Store<EntityStore> store) {
        NPCEntity npc = store.getComponent(npcRef, NPCEntity.getComponentType());
        String wildRole = npc.getRoleName();
        String tamedRole = WILD_TO_TAMED.get(wildRole);

        if (tamedRole == null) return false;

        int roleIndex = RoleBuilder.get().getRoleIndex(tamedRole);
        RoleChangeSystem.requestRoleChange(npcRef, roleIndex);
        return true;
    }
}
```

---

## Asset Structure

### Tamed Role Template

**Server/NPC/Roles/Tamed/_Templates/Template_Animal_Tamed.json:**
```json
{
  "Type": "Abstract",
  "Reference": "Template_Animal_Neutral",
  "Modify": {
    "PlayerDefaultAttitude": "Revered",
    "Timid": false,
    "AttackWhenStartled": false,
    "ChanceToTurnFriendly": 100,
    "FriendlyOverrideTime": 2147483647,
    "WeightFollowItem": 30,
    "WeightFollow": 50,
    "LeashDistance": 30,
    "WanderRadius": 5
  }
}
```

### Animal-Specific Tamed Roles

**Server/NPC/Roles/Tamed/Livestock/Horse_Tamed.json:**
```json
{
  "Type": "Variant",
  "Reference": "Template_Animal_Tamed",
  "Modify": {
    "Appearance": "Horse",
    "FlockArray": ["Horse_Tamed", "Horse_Foal_Tamed"],
    "LovedItems": ["Plant_Crop_Carrot_Item"],
    "IsMountable": true,
    "MountAnchorY": 1.6,
    "MountMovementConfig": "Mount"
  }
}
```

---

## Critical: REVERED Attitude and Mount/Harvest

### The Problem

The base game's mount and harvest interactions only allow these attitudes:
- `Neutral`
- `Friendly`
- `Hostile`

**REVERED is NOT included by default!** This means tamed animals (with REVERED attitude) cannot be mounted or harvested without a patch.

### The Solution

Create a patch to add REVERED to the allowed attitudes:

**Server/Patch/Template_Animal_Neutral_AllowRevered.json:**
```json
{
  "_comment": "Allow Revered attitude for mount and harvest interactions (tamed animals)",
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [
      {
        "_find": { "Enabled": { "Compute": "IsHarvestable" } },
        "Instructions": [
          {
            "_find": { "Sensor": { "Type": "Not" } },
            "Sensor": {
              "Sensor": {
                "Sensors": [
                  {},
                  {
                    "Attitudes": {
                      "Compute": "HarvestRequiredAttitudes"
                    }
                  }
                ]
              }
            }
          }
        ]
      },
      {
        "_find": { "Enabled": { "Compute": "IsMountable" } },
        "Instructions": [
          {
            "_find": { "Sensor": { "Type": "Not" } },
            "Sensor": {
              "Sensor": {
                "Attitudes": ["Neutral", "Friendly", "Hostile", "Revered"]
              }
            }
          }
        ]
      }
    ]
  }
}
```

### Why Not FRIENDLY?

| Attitude | Behavior |
|----------|----------|
| `FRIENDLY` | Temporary - times out via `FriendlyOverrideTime` |
| `REVERED` | Permanent - intended for tamed/owned entities |

Using FRIENDLY causes tamed animals to eventually revert to neutral. REVERED is the correct attitude for permanently tamed animals.

---

## Interaction Hint Patches

### Tame/Feed Hints for Non-Mountable Animals

For animals that don't have mount or harvest interactions (pigs, chickens, etc.), add tame/feed hints:

**Server/Patch/Template_Animal_Neutral_Taming.json:**
```json
{
  "_comment": "Tame/Feed hints for non-mountable/harvestable animals",
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Enabled": {
      "Compute": "IsHarvestable || IsMountable || !isEmptyStringArray(LovedItems)"
    },
    "Instructions": [
      {
        "_op": "add",
        "_index": 0,
        "Enabled": {
          "Compute": "!isEmptyStringArray(LovedItems) && !IsMountable && !IsHarvestable"
        },
        "Instructions": [
          {
            "_comment": "Wild animal = Tame hint",
            "Continue": true,
            "Sensor": { "Type": "Not", "Sensor": { "Type": "Tamed", "Set": true } },
            "Actions": [{
              "Type": "SetInteractable",
              "Interactable": true,
              "Hint": "animalbreeding.interactionHints.tame"
            }]
          },
          {
            "_comment": "Tamed animal = Feed hint",
            "Continue": true,
            "Sensor": { "Type": "Tamed", "Set": true },
            "Actions": [{
              "Type": "SetInteractable",
              "Interactable": true,
              "Hint": "animalbreeding.interactionHints.feed"
            }]
          }
        ]
      }
    ]
  }
}
```

**Important:** Use `!IsMountable && !IsHarvestable` to avoid interfering with mount/harvest interactions on horses and cows.

---

## Setting Attitude at Runtime

When taming via Java code, set the attitude using reflection on `WorldSupport`:

```java
// Get the attitude field (cache this at startup)
Field attitudeField = WorldSupport.class.getDeclaredField("defaultPlayerAttitude");
attitudeField.setAccessible(true);

// In taming code
WorldSupport worldSupport = role.getWorldSupport();
attitudeField.set(worldSupport, Attitude.REVERED);
```

This is necessary because:
1. The ECS `HyTameComponent` tracks taming state
2. The role's `WorldSupport` controls actual NPC behavior
3. Both need to be updated when taming

---

## Initialization Race Conditions

### Problem

ECS systems may initialize before the plugin is fully ready, causing null pointer exceptions:

```java
// This can fail if called during early ECS initialization
LaitsBreedingPlugin.getInstance().getConfigManager() // NPE!
```

### Solution

Add defensive null checks in system constructors:

```java
public HyTameActivateSystem() {
    // ... other initialization ...

    // Defensive null check for early initialization
    LaitsBreedingPlugin plugin = LaitsBreedingPlugin.getInstance();
    if (plugin != null && plugin.getConfigManager() != null) {
        this.validGroups = plugin.getConfigManager().getTameableAnimalGroups();
    } else {
        // Fallback to default groups if plugin not ready
        this.validGroups = Set.of("Livestock", "PreyBig", "Prey");
    }
}
```

### Scheduled Tasks

For scheduled tasks that access game state, add delays and null checks:

```java
scheduledTasks.add(tickScheduler.scheduleAtFixedRate(() -> {
    try {
        // Defensive null checks for early initialization
        if (Universe.get() == null) return;
        var worlds = Universe.get().getWorlds();
        if (worlds == null) return;

        // Safe to proceed
        // ...
    } catch (Exception e) {
        // Silent - may happen during early initialization
    }
}, 2, 5, TimeUnit.SECONDS));  // Delay start by 2 seconds
```

---

## Debugging Tips

### Inspector MCP

Use the Hytale Inspector MCP to examine loaded assets:

```
mcp__inspector__inspector_find_asset
  path: "Template_Animal_Neutral"

mcp__inspector__inspector_get_asset_path
  category: "NPC"
  assetId: "Roles/_Core/Templates/Template_Animal_Neutral"
  path: "content.InteractionInstruction"
```

**Note:** Inspector shows base assets only, not patched versions. Patches are applied at runtime by Hytalor.

### Verify Attitude

Check an entity's attitude in-game:
1. Get the entity reference
2. Access `role.getWorldSupport().getDefaultPlayerAttitude()`
3. Or use sensor checks in behavior trees

---

## Despawn Prevention

### The Problem

Tamed animals need to persist even when players move far away. By default, NPCs despawn when players leave the area.

### Asset-Level Options (Limited)

**Spawn Markers** have a `DeactivationDistance` field that controls despawn distance:

```json
{
  "Model": "NPC_Spawn_Marker",
  "NPCs": [{ "Name": "Cow_Tamed", "Weight": 100 }],
  "DeactivationDistance": 1500
}
```

**However**, spawn markers only work for:
- Pre-placed NPCs in prefabs/structures
- Static, location-bound spawns

They **do NOT work** for dynamically tamed animals because:
1. Spawn markers are placed at fixed world locations
2. Tamed animals are created at runtime from wild animals
3. There's no way to create spawn markers programmatically

See [NPC Spawn Markers](npc-spawn-markers.md) for details.

### Java-Side Solution

For dynamically tamed animals, disable spawn tracking via Java:

```java
// In TameHelper or when taming succeeds
npcEntity.updateSpawnTrackingState(false);
```

This tells the spawn system not to track/despawn the NPC.

**Caveats:**
- Must be called after taming
- May need to be re-applied after certain events (role change, chunk reload)
- Edge cases exist with world save/load

### Recommended Approach

1. **Call `updateSpawnTrackingState(false)`** immediately after taming
2. **Add a periodic check** to ensure it stays disabled for tamed animals
3. **Use persistence system** to restore taming state on world load

```java
// Periodic check (every 30 seconds)
scheduledTasks.add(tickScheduler.scheduleAtFixedRate(() -> {
    for (World world : Universe.get().getWorlds()) {
        world.execute(store -> {
            store.forEachEntity(HyTameComponent.getComponentType(), (ref, hyTame) -> {
                if (hyTame.isTamed()) {
                    NPCEntity npc = store.getComponent(ref, NPCEntity.getComponentType());
                    if (npc != null) {
                        npc.updateSpawnTrackingState(false);
                    }
                }
            });
        });
    }
}, 30, 30, TimeUnit.SECONDS));
```

---

## Related Documentation

- [Hytalor Patch Format](hytalor-patch-format.md)
- [NPC Attitude System](npc-attitude-system.md)
- [NPC Interactions](npc-interactions.md)
- [NPC Interaction Hints](npc-interaction-hints.md)
- [NPC Spawn Markers](npc-spawn-markers.md)
- [ECS Patterns](ecs-patterns.md)
