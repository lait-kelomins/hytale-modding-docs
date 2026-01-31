# NPC Attitude System

> Decompiled from `HytaleServer.jar` - January 2026

## Overview

The attitude system determines how NPCs behave towards players and other NPCs. Attitudes are resolved through a priority-based provider system, allowing multiple mechanisms to influence NPC behavior.

---

## Attitude Enum

**Package:** `com.hypixel.hytale.server.core.asset.type.attitude.Attitude`

| Value | Description |
|-------|-------------|
| `IGNORE` | "is ignoring the target" |
| `HOSTILE` | "is hostile towards the target" |
| `NEUTRAL` | "is neutral towards the target" |
| `FRIENDLY` | "is friendly towards the target" |
| `REVERED` | "reveres the target" (permanent, for tamed entities) |

```java
import com.hypixel.hytale.server.core.asset.type.attitude.Attitude;

// Access all values
Attitude[] allAttitudes = Attitude.VALUES;

// Codec for serialization
EnumCodec<Attitude> codec = Attitude.CODEC;
```

---

## Attitude Resolution

Attitudes are resolved via `AttitudeView`, which checks providers in priority order (lowest first). The first non-null result is used.

### Priority Order

| Priority | Provider | Description |
|----------|----------|-------------|
| 0 | Override Memory | Temporary `overrideAttitude()` calls |
| 100 | Reputation System | Player reputation with NPC's group |
| 200 | Attitude Map | Static NPC group relationships |
| MAX_INT | Default | `defaultPlayerAttitude` or `defaultNPCAttitude` |

**Class:** `com.hypixel.hytale.server.npc.blackboard.view.attitude.AttitudeView`

---

## 3 Ways to Change Attitude

### 1. Temporary Override (WorldSupport)

Override attitude toward a specific entity for a limited duration.

**Class:** `com.hypixel.hytale.server.npc.role.support.WorldSupport`

```java
// Get WorldSupport from Role
WorldSupport ws = role.getWorldSupport();

// Override attitude for 10 seconds
ws.overrideAttitude(targetRef, Attitude.FRIENDLY, 10.0);

// Check if there's an active override
Attitude override = ws.getOverriddenAttitude(targetRef);
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `overrideAttitude(Ref target, Attitude attitude, double duration)` | Set temporary attitude |
| `getOverriddenAttitude(Ref target)` | Get current override (or null) |
| `getDefaultPlayerAttitude()` | Default attitude toward players |
| `getDefaultNPCAttitude()` | Default attitude toward other NPCs |
| `getAttitude(Ref self, Ref target, ComponentAccessor accessor)` | Get resolved attitude |

**Notes:**
- Duration is in seconds
- Overrides are stored in `attitudeOverrideMemory` (Int2ObjectMap)
- Overrides expire automatically via `tick(float dt)`
- Cache clears every 0.1 seconds

---

### 2. Reputation System

Track player reputation with NPC factions/groups. Reputation values map to attitudes via `ReputationRank`.

**Classes:**
- `com.hypixel.hytale.builtin.adventure.reputation.ReputationPlugin`
- `com.hypixel.hytale.builtin.adventure.reputation.assets.ReputationRank`
- `com.hypixel.hytale.builtin.adventure.reputation.assets.ReputationGroup`

#### ReputationRank Structure

```java
public class ReputationRank {
    String id;
    int minValue;      // Inclusive
    int maxValue;      // Exclusive
    Attitude attitude; // Attitude for this rank

    // Check if value falls in this rank
    public boolean containsValue(int value) {
        return value >= minValue && value < maxValue;
    }
}
```

#### Changing Reputation

```java
ReputationPlugin rep = ReputationPlugin.get();

// Change reputation by value (returns new reputation value)
int newRep = rep.changeReputation(player, npcRef, +50, accessor);

// Or by group ID directly
int newRep = rep.changeReputation(player, "villagers", +50, accessor);

// Get current reputation value
int repValue = rep.getReputationValue(store, playerRef, npcRef);

// Get reputation rank
ReputationRank rank = rep.getReputationRank(store, playerRef, npcRef);

// Get attitude from reputation
Attitude attitude = rep.getAttitude(store, playerRef, npcRef);
```

#### Storage Types

Configurable via `ReputationGameplayConfig`:

| Type | Description |
|------|-------------|
| `PerPlayer` | Each player has their own reputation |
| `PerWorld` | Reputation is shared across all players |

#### Asset Paths

- Reputation Ranks: `NPC/Reputation/Ranks/*.json`
- Reputation Groups: `NPC/Reputation/Groups/*.json`

#### Example Rank JSON (inferred structure)

```json
{
    "Id": "Hostile",
    "MinValue": -1000,
    "MaxValue": -100,
    "Attitude": "HOSTILE"
}
```

---

### 3. Attitude Groups (Static Relationships)

Define which NPC groups have which attitude toward other groups.

**Classes:**
- `com.hypixel.hytale.server.npc.blackboard.view.attitude.AttitudeMap`
- `com.hypixel.hytale.server.npc.config.AttitudeGroup`

```java
// Get attitude from static group mappings
AttitudeMap map = NPCPlugin.get().getAttitudeMap();
Attitude attitude = map.getAttitude(role, targetRef, accessor);
```

The `AttitudeMap` uses NPC group tags to determine relationships:
- Each NPC has an `attitudeGroup` index
- Groups define which other groups they're hostile/friendly/etc. toward
- Special handling for `self` (same role) and `player` groups

---

## ActionOverrideAttitude

Behavior tree action that overrides attitude toward a sensor target.

**Class:** `com.hypixel.hytale.server.npc.corecomponents.entity.ActionOverrideAttitude`

```java
public class ActionOverrideAttitude extends ActionBase {
    protected final Attitude attitude;
    protected final double duration;

    @Override
    public boolean execute(...) {
        Ref<EntityStore> target = sensorInfo.getPositionProvider().getTarget();
        role.getWorldSupport().overrideAttitude(target, this.attitude, this.duration);
        return true;
    }
}
```

Use this in NPC behavior definitions to change attitude based on game events.

---

## ReputationAttitudeSystem

ECS system that connects reputation to the attitude provider chain.

**Class:** `com.hypixel.hytale.builtin.adventure.npcreputation.ReputationAttitudeSystem`

Registers a provider at priority 100 that returns attitude based on player reputation:

```java
view.registerProvider(100, (ref, role, targetRef, accessor) -> {
    Player playerComponent = store.getComponent(targetRef, Player.getComponentType());
    if (playerComponent == null) {
        return null;  // Not a player, skip
    }
    return ReputationPlugin.get().getAttitude(store, targetRef, ref);
});
```

---

## Related Classes

| Class | Package | Purpose |
|-------|---------|---------|
| `Attitude` | `server.core.asset.type.attitude` | Enum of attitude values |
| `WorldSupport` | `server.npc.role.support` | Per-NPC attitude state |
| `AttitudeView` | `server.npc.blackboard.view.attitude` | Resolves attitude via providers |
| `IAttitudeProvider` | `server.npc.blackboard.view.attitude` | Provider interface |
| `AttitudeMap` | `server.npc.blackboard.view.attitude` | NPC group relationships |
| `AttitudeGroup` | `server.npc.config` | Group attitude config asset |
| `AttitudeMemoryEntry` | `server.npc.util` | Stores override with expiry |
| `ReputationPlugin` | `builtin.adventure.reputation` | Reputation tracking |
| `ReputationRank` | `builtin.adventure.reputation.assets` | Value-to-attitude mapping |
| `ReputationGroup` | `builtin.adventure.reputation.assets` | Faction/group definition |
| `ReputationGroupComponent` | `builtin.adventure.reputation` | NPC component for group membership |
| `ActionOverrideAttitude` | `server.npc.corecomponents.entity` | Behavior action |
| `EntityFilterAttitude` | `server.npc.corecomponents.entity.filters` | Filter entities by attitude |

---

## Usage Examples

### Make NPC friendly to player temporarily

```java
Role role = npcEntity.getRole();
WorldSupport ws = role.getWorldSupport();

// Friendly for 30 seconds
ws.overrideAttitude(playerRef, Attitude.FRIENDLY, 30.0);
```

### Check NPC's attitude toward player

```java
WorldSupport ws = role.getWorldSupport();
Attitude attitude = ws.getAttitude(npcRef, playerRef, store);

if (attitude == Attitude.HOSTILE) {
    // NPC will attack
} else if (attitude == Attitude.FRIENDLY || attitude == Attitude.REVERED) {
    // NPC is friendly
}
```

### Increase player reputation with faction

```java
ReputationPlugin rep = ReputationPlugin.get();

// Player helped the villagers
rep.changeReputation(player, "villagers", +100, accessor);

// Player killed a villager
rep.changeReputation(player, "villagers", -200, accessor);
```

---

## Decompiled Source Files

Located in `tools/reverse-engineer/decompiled-src/`:

- `com/hypixel/hytale/server/core/asset/type/attitude/Attitude.java`
- `com/hypixel/hytale/server/npc/role/support/WorldSupport.java`
- `com/hypixel/hytale/server/npc/blackboard/view/attitude/AttitudeView.java`
- `com/hypixel/hytale/server/npc/blackboard/view/attitude/AttitudeMap.java`
- `com/hypixel/hytale/server/npc/corecomponents/entity/ActionOverrideAttitude.java`
- `com/hypixel/hytale/builtin/adventure/reputation/ReputationPlugin.java`
- `com/hypixel/hytale/builtin/adventure/reputation/assets/ReputationRank.java`
- `com/hypixel/hytale/builtin/adventure/npcreputation/ReputationAttitudeSystem.java`

---

## REVERED vs FRIENDLY for Tamed Animals

When implementing taming systems, use **REVERED** instead of **FRIENDLY**:

| Attitude | Behavior | Use Case |
|----------|----------|----------|
| `FRIENDLY` | Temporary - times out via `FriendlyOverrideTime` parameter | Food attraction |
| `REVERED` | Permanent - does not time out | Tamed/owned animals |

### Gotcha: Mount/Harvest Blocked by REVERED

The base game's `Template_Animal_Neutral` only allows these attitudes for mount/harvest interactions:
- `Neutral`, `Friendly`, `Hostile`

**REVERED is NOT included by default!** Tamed animals with REVERED attitude cannot be mounted or harvested without patching the interaction sensors.

**Solution:** Patch the `CanInteract` sensor to include REVERED:
```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [{
      "_find": { "Enabled": { "Compute": "IsMountable" } },
      "Instructions": [{
        "_find": { "Sensor": { "Type": "Not" } },
        "Sensor": {
          "Sensor": {
            "Attitudes": ["Neutral", "Friendly", "Hostile", "Revered"]
          }
        }
      }]
    }]
  }
}
```

See [Asset-Based Taming](asset-based-taming.md) for complete implementation details
