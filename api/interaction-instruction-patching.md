# InteractionInstruction Patching Guide

Reliable patterns for patching NPC interaction behavior via Hytalor.

> **Last Updated:** 2026-02-03 - Major corrections based on testing

---

## Overview

`InteractionInstruction` controls:
1. **Hints** - What text shows when looking at an NPC ("Press [F] to Mount")
2. **Interactability** - Whether F key prompt appears
3. **Actions** - What happens when player interacts (mount, harvest state, etc.)

---

## CRITICAL: Instruction Evaluation Model

### The Single-Instruction Rule

**All logic for a single interaction type MUST be in ONE instruction.** The game evaluates each instruction as a complete unit.

```json
// ✅ CORRECT: Everything in one instruction
{
    "Sensor": {
        "Type": "And",
        "Sensors": [
            { "Type": "Tamed", "Set": true },
            { "Type": "Player", "Range": 5, "Filters": [{ "Type": "ItemInHand", "Items": {...} }] },
            { "Type": "HasInteracted" }   // <-- HasInteracted INSIDE the And
        ]
    },
    "Actions": [
        { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
        { "Type": "HyTameFeedInteraction" }   // <-- Action with SetInteractable
    ]
}

// ❌ WRONG: Separate instructions for hint vs action
// Instruction 1: SetInteractable (conditions without HasInteracted)
// Instruction 2: HasInteracted → Action
// This pattern DOES NOT WORK reliably!
```

### Why Separate Instructions Fail

We discovered through extensive testing that:
1. **Hint instructions** (SetInteractable) and **action instructions** (HasInteracted → Action) cannot be split
2. `HasInteracted` sensors in separate instructions do not reliably trigger when conditions are met
3. The game may evaluate instructions independently, causing race conditions

### Correct Pattern Summary

| Element | Location |
|---------|----------|
| Condition sensors (Tamed, Player, etc.) | Inside `And` in main Sensor |
| `HasInteracted` sensor | Inside same `And` with conditions |
| `SetInteractable` action | In Actions array |
| Custom action (HyTameFeedInteraction) | In same Actions array |

---

## Detecting Player Held Items

Use `Player` sensor with `ItemInHand` filter:

```json
{
  "Type": "Player",
  "Range": 5,
  "Filters": [
    {
      "Type": "ItemInHand",
      "Items": { "Compute": "LovedItems" }
    }
  ]
}
```

**Works in:** InteractionInstruction context (combined with HasInteracted in same And sensor)

---

## Instruction Flow

Instructions are evaluated in order. Key behaviors:

| Field | Behavior |
|-------|----------|
| `Continue: true` | After matching, continue to next instruction |
| No `Continue` | Stop processing after this instruction matches |
| `Sensor` | Condition that must match for Actions to run |
| `Enabled` | Compute expression that enables/disables entire instruction |

### ⚠️ Important: Continue and Custom Actions

When using `Continue: true` with custom actions like `HyTameFeedInteraction`, the action still fires but processing continues to subsequent instructions. This is useful when you want your action to execute but also allow fallback behavior (like mounting) when conditions aren't fully met.

---

## Working Patterns

### Pattern 1: Taming Wild Animals (Mountable)

```json
{
    "Continue": true,
    "Enabled": { "Compute": "!isEmptyStringArray(LovedItems)" },
    "Sensor": {
        "Type": "And",
        "Sensors": [
            { "Type": "Not", "Sensor": { "Type": "Tamed", "Set": true } },
            {
                "Type": "Player",
                "Range": 5,
                "Filters": [{ "Type": "ItemInHand", "Items": { "Compute": "LovedItems" } }]
            },
            { "Type": "HasInteracted" }
        ]
    },
    "Actions": [
        { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
        { "Type": "HyTameFeedInteraction" }
    ]
}
```

**Key points:**
- `Not Tamed` + `Player holding LovedItems` + `HasInteracted` all in same `And`
- Both `SetInteractable` and custom action in same `Actions` array
- `Continue: true` allows mount to still work when player has empty hands

### Pattern 2: Breeding Tamed Animals (Mountable)

```json
{
    "Continue": true,
    "Enabled": { "Compute": "!isEmptyStringArray(LovedItems)" },
    "Sensor": {
        "Type": "And",
        "Sensors": [
            { "Type": "Tamed", "Set": true },
            {
                "Type": "Player",
                "Range": 5,
                "Filters": [{ "Type": "ItemInHand", "Items": { "Compute": "LovedItems" } }]
            },
            { "Type": "HasInteracted" }
        ]
    },
    "Actions": [
        { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
        { "Type": "HyTameFeedInteraction" }
    ]
}
```

### Pattern 3: Blocking Mount When Holding Items

**NOT NEEDED** when using the patterns above with `Continue: true`. The HyTameFeedInteraction fires when holding items, and mount fires when not holding items. No explicit blocking required.

---

## Common Sensors

| Sensor | Purpose | Notes |
|--------|---------|-------|
| `Tamed` | Check if NPC is tamed (`Set: true/false`) | Works with HyTame component |
| `HasInteracted` | Player pressed F key | **Must be inside And with conditions** |
| `CanInteract` | Player can interact based on attitudes | Uses attitude list |
| `Alarm` | Check alarm state (`Unset`, `Set`, `Passed`) | For cooldowns |
| `InteractionContext` | Check tool context (shears, bucket, etc.) | Left-click interactions |
| `Player` | Detect nearby player with filters | Use with `ItemInHand` filter |
| `Any` | Always matches | Useful for default cases |
| `Not`, `And`, `Or` | Logical operators | `Not` works inside `And` ✓ |

### Sensor Combinations That Work

| Combination | Works? | Notes |
|-------------|--------|-------|
| `And[Tamed, Player, HasInteracted]` | ✅ | Standard pattern |
| `And[Not(Tamed), Player, HasInteracted]` | ✅ | Wild animal detection |
| `And[HasInteracted, Not(Player)]` | ✅ | Mount when NOT holding items |
| Separate instruction with just `HasInteracted` | ❌ | Does not trigger reliably |

---

## Hytalor Array Patching

### Selection Methods

| Method | Use Case |
|--------|----------|
| `_index` | Position-based (fragile if other patches shift indices) |
| `_find` | JsonPath query for first match (recommended) |
| `_findAll` | JsonPath query for all matches |

### Operations

| Operation | Purpose |
|-----------|---------|
| `merge` | Merge fields into element (default) |
| `replace` | Replace entire element |
| `add` | Insert new element |
| `remove` | Delete element |

### _find Object Matching

`_find` uses object matching to locate array elements by field values:

```json
// Find block by Enabled.Compute value
"_find": {
  "Enabled": { "Compute": "IsMountable" }
}

// Find instruction by sensor type
"_find": {
  "Sensor": { "Type": "HasInteracted" }
}

// Find by action type
"_find": {
  "Actions": [{ "Type": "State", "State": "$Harvest" }]
}
```

This is more robust than `_index` because it doesn't depend on array position.

---

## Template_Animal_Neutral Structure

The `InteractionInstruction.Instructions` array has these blocks:

| Index | Enabled | Purpose |
|-------|---------|---------|
| 0 | `!isEmptyStringArray(LovedItems) && !IsMountable && !IsHarvestable` | Simple tameable animals |
| 1 | `IsHarvestable` | Harvestable animals (sheep, etc.) |
| 2 | `IsMountable` | Mountable animals (horses, etc.) |

### Mountable Block Structure (with taming/breeding patches)

After applying taming and breeding patches correctly:

| Idx | Purpose | Sensor Pattern |
|-----|---------|----------------|
| 0 | Taming (wild + LovedItems) | `And[Not(Tamed), Player(ItemInHand), HasInteracted]` |
| 1 | Breeding (tamed + LovedItems) | `And[Tamed, Player(ItemInHand), HasInteracted]` |
| 2 | Gate: disable if can't interact | `Not(CanInteract)` |
| 3 | Show mount hint | `Any` with Continue |
| 4 | Mount action | `HasInteracted` (vanilla) |

**Note:** Our taming/breeding instructions use `Continue: true`, so when the player doesn't have LovedItems, processing continues to the vanilla mount instruction.

---

## Hytalor Patch Patterns

### 1. Adding instruction at start of block (using _find)

```json
{
    "_find": { "Enabled": { "Compute": "IsMountable" } },
    "Instructions": [
        { "_op": "addBefore", "_index": 0, ... }
    ]
}
```

### 2. Finding and modifying nested instruction

```json
{
    "_find": { "Enabled": { "Compute": "IsMountable" } },
    "Instructions": [
        {
            "_find": { "Sensor": { "Type": "Not" } },
            "Sensor": { "Sensor": { "Attitudes": ["Neutral", "Friendly", "Hostile", "Revered"] } }
        }
    ]
}
```

### 3. Complete working taming patch

```json
{
    "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
    "InteractionInstruction": {
        "Instructions": [
            {
                "_find": { "Enabled": { "Compute": "IsMountable" } },
                "Instructions": [
                    {
                        "_op": "addBefore",
                        "_index": 0,
                        "Enabled": { "Compute": "!isEmptyStringArray(LovedItems)" },
                        "Continue": true,
                        "Sensor": {
                            "Type": "And",
                            "Sensors": [
                                { "Type": "Not", "Sensor": { "Type": "Tamed", "Set": true } },
                                { "Type": "Player", "Range": 5, "Filters": [
                                    { "Type": "ItemInHand", "Items": { "Compute": "LovedItems" } }
                                ]},
                                { "Type": "HasInteracted" }
                            ]
                        },
                        "Actions": [
                            { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
                            { "Type": "HyTameFeedInteraction" }
                        ]
                    }
                ]
            }
        ]
    }
}
```

---

## Debugging Tips

1. Use inspector to view patched asset state: `inspector_get_asset_path`
2. Publish patches live with `inspector_publish_patch`
3. Refresh with `inspector_refresh_assets`
4. Check array lengths changed as expected
5. Verify sensor conditions with live entity testing
6. **Check entity components:** Use `inspector_get_entity` to verify HyTame component state

---

## Common Mistakes (Lessons Learned)

### ❌ Mistake 1: Separate Hint and Action Instructions

```json
// WRONG - This does NOT work!
{ "Sensor": { "And": [Tamed, Player(Item)] }, "Actions": [SetInteractable] },
{ "Sensor": { "HasInteracted" }, "Actions": [CustomAction] }
```

**Why it fails:** The HasInteracted instruction fires independently and doesn't respect the conditions from the previous instruction.

### ❌ Mistake 2: Trying to Block with Empty Actions

```json
// WRONG - Empty actions don't block subsequent instructions
{ "Sensor": { "And": [Tamed, Player(Item)] }, "Actions": [] }
```

**Why it fails:** Instructions with empty Actions don't consume the interaction event.

### ❌ Mistake 3: Complex And+Not Without HasInteracted

```json
// WRONG - Conditions without HasInteracted don't trigger actions
{ "Sensor": { "And": [Not(Tamed), Player(Item)] }, "Actions": [CustomAction] }
```

**Why it fails:** Without HasInteracted in the sensor, the action conditions are evaluated but the action doesn't fire on F-key press.

### ✅ Correct Pattern

```json
{
    "Sensor": {
        "Type": "And",
        "Sensors": [
            { "Type": "Not", "Sensor": { "Type": "Tamed", "Set": true } },
            { "Type": "Player", "Range": 5, "Filters": [{ "Type": "ItemInHand", ... }] },
            { "Type": "HasInteracted" }  // <-- Required!
        ]
    },
    "Actions": [
        { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
        { "Type": "CustomAction" }  // <-- In same Actions array!
    ]
}
```

---

## Related Documentation

- [NPC Interactions Reference](npc-interactions.md)
- [Hytalor Patch Format](hytalor-patch-format.md)
- [NPC Attitude System](npc-attitude-system.md)
