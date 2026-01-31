# Hytalor Patch Format

**Source:** [HypersonicSharkz/Hytalor](https://github.com/HypersonicSharkz/Hytalor)

## Overview

Hytalor is a lightweight asset patching framework for Hytale that enables multiple mods to modify the same base game assets without overwriting each other. Rather than replacing entire JSON files, it applies smaller patches that merge together at runtime.

## How It Works

1. Gathers all patches targeting a specific base asset
2. Copies the base asset and sequentially applies each patch
3. Writes the combined result into the Hytalor AssetPack (loads last)
4. Hot-reloads when patch files change

## File Structure

Patch files must be in `Server/Patch/` within your plugin:

```
YourPlugin/
├── manifest.json
└── Server/
    └── Patch/
        └── YourPatch.json
```

## Basic Patch Format

**IMPORTANT:** Properties go directly at root level - there is NO `"Modify": { }` wrapper!

```json
{
  "BaseAssetPath": "path/to/asset",
  "PropertyName": "new value",
  "NestedObject": {
    "SubProperty": "value"
  }
}
```

### Example: Simple Property Patch

```json
{
  "BaseAssetPath": "Weathers/Zone1/Zone1_Sunny",
  "Stars": "Sky/Void.png"
}
```

### Example: Item Interaction Patch

```json
{
  "$Comment": "Add FeedAnimal interaction to crop items",
  "BaseAssetPath": "Item/Items/Plant/Crop/_Template/Template_Crop_Item",
  "Interactions": {
    "Use": "Root_FeedAnimal",
    "Secondary": "Root_Secondary_Consume_Food_T1"
  }
}
```

## Wildcard Targeting

Apply a single patch to multiple assets:

```json
{
  "BaseAssetPath": "Weathers/Zone1/*",
  "Stars": "Sky/Void.png"
}
```

## Array Operations

Arrays use special control fields for modification:

### Control Fields

| Field | Purpose |
|-------|---------|
| `_index` | Select element by position (0-based) |
| `_find` | Select first matching element |
| `_findAll` | Select all matching elements |
| `_op` | Specify the operation |

### Operations

| Operation | Function |
|-----------|----------|
| `merge` | Update specified fields (default) |
| `replace` | Replace entire element |
| `add` | Insert new element at end |
| `addBefore` | Insert before target element |
| `addAfter` | Insert after target element |
| `upsert` | Merge if exists, otherwise add |
| `remove` | Delete element |

### Example: Merge Array Element

Base asset:
```json
{
  "Clouds": [
    { "Texture": "Sky/Clouds/Light.png", "Speed": 0.5, "Opacity": 0.8 }
  ]
}
```

Patch (changes only Speed):
```json
{
  "BaseAssetPath": "Weathers/Zone1/Zone1_Sunny",
  "Clouds": [
    { "_index": 0, "_op": "merge", "Speed": 0.7 }
  ]
}
```

### Example: Add New Array Element

```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [
      {
        "_op": "add",
        "Enabled": { "Compute": "SomeCondition" },
        "Instructions": [ ... ]
      }
    ]
  }
}
```

### Example: Find and Replace

```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [
      {
        "_find": { "Enabled": { "Compute": "IsMountable" } },
        "_op": "replace",
        "Enabled": { "Compute": "IsMountable" },
        "Instructions": [ ... new instructions ... ]
      }
    ]
  }
}
```

### Example: Upsert (Merge or Add)

```json
{
  "Categories": [
    {
      "_op": "upsert",
      "_find": { "Id": "Arcane_Spellbooks" },
      "Icon": "Custom.Icon"
    }
  ]
}
```

## Primitive Array Operations

For arrays of simple values (strings, numbers):

### Add Value

```json
{
  "BaseAssetPath": "Item/Items/Bench/Bench_Arcane",
  "Categories": [
    { "_op": "add", "_value": "Custom.Category" }
  ]
}
```

### Replace Entire Array

Use JsonPath syntax:
```json
{
  "BaseAssetPath": "Item/Items/Bench/Bench_Arcane",
  "$.Tags.Type": ["Custom.Type"]
}
```

## JsonPath Queries

Hytalor supports JsonPath for complex selections:

### Direct JsonPath Assignment

```json
{
  "BaseAssetPath": "Weathers/Zone1/*",
  "$.Clouds[*].Colors[?(@.Hour < 12)].Color": "#00EE00"
}
```

### Structured JsonPath

```json
{
  "BaseAssetPath": "Weathers/Zone1/*",
  "Clouds": [
    {
      "_findAll": "$[*]",
      "Colors": [
        {
          "_findAll": "$[?(@.Hour < 12)]",
          "Color": "#00FF00"
        }
      ]
    }
  ]
}
```

## Common Patterns

### Adding NPC Interaction Instructions

```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Enabled": true,
    "Instructions": [
      {
        "_op": "add",
        "Enabled": { "Compute": "!isEmptyStringArray(LovedItems)" },
        "Instructions": [
          {
            "Sensor": { "Type": "CanInteract", "Attitudes": ["Neutral"] },
            "Actions": [
              { "Type": "SetInteractable", "Interactable": true, "Hint": "key" }
            ]
          }
        ]
      }
    ]
  }
}
```

### Modifying Existing Instruction

```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [
      {
        "_find": { "Enabled": { "Compute": "IsMountable" } },
        "_op": "replace",
        "Enabled": { "Compute": "IsMountable" },
        "Instructions": [ ... modified instructions ... ]
      }
    ]
  }
}
```

## Nested _find for Deep Modifications

When modifying deeply nested structures, chain `_find` operations:

### Example: Modify Nested Sensor Attitudes

The mount interaction has this structure:
```json
{
  "Instructions": [{
    "Enabled": { "Compute": "IsMountable" },
    "Instructions": [{
      "Sensor": {
        "Type": "Not",
        "Sensor": {
          "Attitudes": ["Neutral", "Friendly", "Hostile"]
        }
      }
    }]
  }]
}
```

To add "Revered" to the attitudes:

```json
{
  "BaseAssetPath": "NPC/Roles/_Core/Templates/Template_Animal_Neutral",
  "InteractionInstruction": {
    "Instructions": [
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

**How it works:**
1. First `_find` locates the instruction block with `IsMountable`
2. Second `_find` locates the sensor with `Type: "Not"` within that block
3. The `Sensor.Sensor.Attitudes` array is replaced with the new value

---

## Key Points

1. **No `Modify` wrapper** - Properties go directly at root level
2. **Merge is default** - Nested objects merge, don't replace
3. **Array ops need `_op`** - Use control fields for array modifications
4. **Wildcards work** - Use `*` in BaseAssetPath for multi-asset patches
5. **Hot-reload** - Changes apply without restart
6. **Load order** - Hytalor AssetPack loads last, overriding originals
7. **No `$` prefix in keys** - Keys starting with `$` are treated as JsonPath queries. Use `_comment` instead of `$Comment` for annotations
8. **Nested _find** - Chain `_find` operations to modify deeply nested structures

## Related Documentation

- [NPC Interaction Hints](npc-interaction-hints.md)
- [Item Food Templates](item-food-templates.md)
- [Assets API](assets-api.md)
