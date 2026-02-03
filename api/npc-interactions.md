# NPC Interactions Reference

> **Source Asset:** `NPC/Roles/_Core/Templates/Template_Animal_Neutral` (default/vanilla)
> **Last Updated:** 2026-01-30

## Quick Reference

| Location | Purpose | Key Parameters |
|----------|---------|----------------|
| `content.InteractionVars` | Combat/damage interactions | `Melee_Damage`, `Parent`, `DamageCalculator` |
| `content.InteractionInstruction` | Player-triggered interactions | Harvesting, Mounting |
| `content.Instructions[4].$Harvest` | Harvest execution logic | Drops, sounds, particles, model swaps |

---

## 1. InteractionVars

Path: `content.InteractionVars`

Combat and damage interaction definitions.

```json
{
  "Melee_Damage": {
    "Interactions": [
      {
        "Parent": "NPC_Attack_Melee_Damage",
        "DamageCalculator": {
          "Type": "Absolute",
          "BaseDamage": { "Physical": 5 },
          "RandomPercentageModifier": 0.1
        }
      }
    ]
  }
}
```

### Key Fields
- `Parent` - References a base interaction definition
- `DamageCalculator.Type` - `Absolute` or other calculation types
- `DamageCalculator.BaseDamage` - Damage values by type (Physical, etc.)

---

## 2. InteractionInstruction

Path: `content.InteractionInstruction`

Player-facing interaction logic (F-key prompts, left-click actions).

### Structure
```json
{
  "Enabled": { "Compute": "IsHarvestable || IsMountable" },
  "Instructions": [
    { /* Harvesting block */ },
    { /* Mounting block */ }
  ]
}
```

### Harvesting Interactions

**Enabled when:** `IsHarvestable` parameter is true

**Logic flow:**
1. Check if NOT harvestable → `SetInteractable: false`
2. Allow contextual interactions but hide F-key prompt (`ShowPrompt: false`)
3. On left-click with tool matching `HarvestInteractionContext` → transition to `$Harvest` state

**Key sensors:**
- `Alarm` (Name: `Harvest_Ready`, State: `Unset`) - Cooldown check
- `CanInteract` with `HarvestRequiredAttitudes` - Attitude check
- `InteractionContext` with `HarvestInteractionContext` - Tool requirement

**Key actions:**
- `SetInteractable` - Enable/disable interaction
- `State` → `$Harvest` - Trigger harvest

### Mounting Interactions

**Enabled when:** `IsMountable` parameter is true

**Requirements:**
- Player can interact (`CanInteract` sensor)
- Player attitude must be `Neutral`, `Friendly`, or `Hostile`

**Key fields:**
- `Hint`: `server.interactionHints.mount`
- `ViewSector`: 360 (can interact from any angle)

**Mount action parameters:**
- `AnchorX/Y/Z` - Mount position offset (from parameters)
- `MovementConfig` - Movement behavior when mounted

> **Note:** Default asset does NOT require taming. Taming requirements are added via patches (e.g., breeding/taming mod).

> **IMPORTANT:** The default attitude list does NOT include `Revered`. If you use REVERED attitude for tamed animals, you must patch the mount interaction to allow it. See [Asset-Based Taming](asset-based-taming.md#critical-revered-attitude-and-mountharvest).

---

## 3. $Harvest State (Instructions)

Path: `content.Instructions[4]` (State: `$Harvest`)

Executes when harvest interaction is triggered.

### Sequence
1. Play `Alerted` animation
2. Spawn particles (`HarvestParticles`)
3. Play sound (`HarvestSound`)
4. Set cooldown alarm (`Harvest_Ready` with `HarvestTimeout`)
5. Drop items from `HarvestDropList` (if set)
6. Handle bucket replacement for milking (if `HarvestAddItemBucket` set)
7. Update model attachments (if `HarvestModelSlot` set)
8. Wait 2-3 seconds → return to `Idle`

### Relevant Parameters
| Parameter | Purpose |
|-----------|---------|
| `HarvestDropList` | Items to drop on harvest |
| `HarvestTimeout` | Cooldown duration range |
| `HarvestParticles` | Particle effect to spawn |
| `HarvestSound` | Sound event to play |
| `HarvestAddItemBucket` | Item to give when player holds `Container_Bucket` |
| `HarvestAddItemDecoBucket` | Item to give when player holds `Deco_Bucket` |
| `HarvestModelSlot` | Model slot to modify |
| `HarvestModelAttachmentHarvested` | Attachment after harvest |
| `HarvestModelAttachmentHarvestable` | Attachment when ready |
| `HarvestInteractionContext` | Tool/context required to harvest |
| `HarvestRequiredAttitudes` | Attitudes that allow harvesting |

---

## Inspector MCP Usage

### Connect
```
mcp__inspector__inspector_connect
```

### Find the Asset
```
mcp__inspector__inspector_find_asset
  path: "Template_Animal_Neutral"
```
Returns: `category: NPC`, `assetId: Roles/_Core/Templates/Template_Animal_Neutral`

### Read Specific Paths
```
mcp__inspector__inspector_get_asset_path
  category: "NPC"
  assetId: "Roles/_Core/Templates/Template_Animal_Neutral"
  path: "content.InteractionVars"
```

```
mcp__inspector__inspector_get_asset_path
  category: "NPC"
  assetId: "Roles/_Core/Templates/Template_Animal_Neutral"
  path: "content.InteractionInstruction"
```

### Generate a Patch
```
mcp__inspector__inspector_generate_patch
  category: "NPC"
  assetId: "Roles/_Core/Templates/Template_Animal_Neutral"
  originalJson: "<original>"
  modifiedJson: "<modified>"
```

### Publish Patch
```
mcp__inspector__inspector_publish_patch
  filename: "my-interaction-patch"
  patchJson: "<patch content>"
```

---

## Related Assets

- `NPC_Attack_Melee_Damage` - Base melee attack interaction
- `Component_Sensor_Standard_Detection` - Detection sensor component
- `Component_Instruction_Damage_Check` - Damage handling component

---

## Notes

- Harvesting uses `InteractionContext` for tool-based triggers (left-click with tool)
- Mounting uses `HasInteracted` sensor (F-key interaction)
- Both require appropriate attitude checks
- The `$Harvest` state name starts with `$` indicating it's a special/internal state
- Model attachments allow visual state changes (e.g., wool on/off for sheep)

---

## Adding Custom Interactions (Taming/Breeding)

> **See:** [InteractionInstruction Patching Guide](interaction-instruction-patching.md) for detailed patterns

### Critical Rule: Single-Instruction Pattern

When adding custom F-key interactions (like taming or breeding), **all logic must be in ONE instruction**:

```json
{
    "Continue": true,
    "Sensor": {
        "Type": "And",
        "Sensors": [
            { "Type": "Tamed", "Set": true },           // Condition
            { "Type": "Player", "Range": 5, "Filters": [{ "Type": "ItemInHand", ... }] },
            { "Type": "HasInteracted" }                  // F-key trigger
        ]
    },
    "Actions": [
        { "Type": "SetInteractable", "Interactable": true, "ShowPrompt": true },
        { "Type": "HyTameFeedInteraction" }              // Custom action
    ]
}
```

**Do NOT use separate instructions** for hint display and action trigger - this pattern fails.
