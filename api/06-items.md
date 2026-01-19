# Hytale Items & Inventory

> **Index:** [Item System](#item-system) | [Item Types](#item-types) | [Crafting](#crafting) | [Inventory](#inventory)

---

## Item System

**Key Packages:**
```
com.hypixel.hytale.server.core.asset.type.item/
com.hypixel.hytale.server.core.asset.type.item.config/
```

---

## Item Configuration

Items are defined as assets with various properties.

### Item Class

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.Item`

### Item Properties

| Class | Purpose |
|-------|---------|
| `ItemCategory` | Item categorization |
| `ItemQuality` | Quality levels |
| `ItemDrop` | Drop definitions |
| `ItemDropList` | Multiple drops |
| `ItemEntityConfig` | Item entity behavior |

---

## Item Types

### Weapons

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.ItemWeapon`

### Tools

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.ItemTool`

Properties:
- `DurabilityLossBlockTypes` - What blocks affect durability

### Armor

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.ItemArmor`

Properties:
- `InteractionModifierId` - Interaction modifiers

### Utility Items

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.ItemUtility`

### Gliders

**Class:** `com.hypixel.hytale.server.core.asset.type.item.config.ItemGlider`

---

## Item Features

### Tool Specs

**Class:** `ItemToolSpec` - Tool capabilities and stats

### Durability

**Class:** `ItemDurabilityConfig` (in gameplay package)

### Pullback/Charge

**Class:** `ItemPullbackConfig` - Bow-style charge mechanics

### Reticle

**Class:** `ItemReticleConfig` - Crosshair/reticle settings

---

## Crafting System

**Package:** `com.hypixel.hytale.server.core.asset.type.item.config`

### Recipe

**Class:** `CraftingRecipe`

### Crafting Events

```java
// ECS Events for crafting
CraftRecipeEvent      // Base crafting event
CraftRecipeEvent.Pre  // Before craft
CraftRecipeEvent.Post // After craft
```

### Crafting Benches

**Package:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.bench`

| Class | Purpose |
|-------|---------|
| `Bench` | Base bench |
| `CraftingBench` | Standard crafting |
| `ProcessingBench` | Processing/smelting |
| `DiagramCraftingBench` | Pattern-based |
| `StructuralCraftingBench` | Building structures |

---

## Drop System

### Drop Containers

**Package:** `com.hypixel.hytale.server.core.asset.type.item.config.container`

| Class | Purpose |
|-------|---------|
| `ItemDropContainer` | Base drop container |
| `SingleItemDropContainer` | Single item drop |
| `MultipleItemDropContainer` | Multiple items |
| `ChoiceItemDropContainer` | Random choice |
| `DroplistItemDropContainer` | From droplist |
| `EmptyItemDropContainer` | No drop |

### Drop Commands

**Class:** `DroplistCommand` - Manage droplists

---

## Inventory

**Package:** `com.hypixel.hytale.server.core.inventory`

### ItemStack

**Class:** `com.hypixel.hytale.server.core.inventory.ItemStack`

The core class representing a stack of items.

| Inner Class | Purpose |
|-------------|---------|
| `ItemStack.Metadata` | Item metadata |

### Item Containers

**Package:** `com.hypixel.hytale.server.core.inventory.container`

| Class | Purpose |
|-------|---------|
| `ItemContainer` | Base container interface |
| `SimpleItemContainer` | Simple implementation |
| `ItemStackItemContainer` | ItemStack-based container |
| `CombinedItemContainer` | Multiple containers combined |
| `DelegateItemContainer` | Delegation pattern |
| `EmptyItemContainer` | Empty container |

### Container Utilities

| Class | Purpose |
|-------|---------|
| `ItemContainerUtil` | Container utility methods |
| `InternalContainerUtilItemStack` | ItemStack utilities |
| `InternalContainerUtilMaterial` | Material utilities |
| `InternalContainerUtilResource` | Resource utilities |

### Slot Filters

**Package:** `com.hypixel.hytale.server.core.inventory.container.filter`

| Class | Purpose |
|-------|---------|
| `ItemSlotFilter` | Base slot filter |
| `ArmorSlotAddFilter` | Armor slot restrictions |
| `NoDuplicateFilter` | Prevent duplicates |
| `ResourceFilter` | Resource type filter |
| `TagFilter` | Tag-based filter |

### Inventory Transactions

**Package:** `com.hypixel.hytale.server.core.inventory.transaction`

| Class | Purpose |
|-------|---------|
| `ItemStackTransaction` | Item transactions |
| `ItemStackSlotTransaction` | Slot-specific transactions |

### Active Slot / Hotbar

| Class | Purpose |
|-------|---------|
| `HotbarManager` | Player hotbar management |
| `SwitchActiveSlotEvent` | Slot change event |
| `SetActiveSlot` (packet) | Network packet |
| `ChangeActiveSlotInteraction` | Slot change interaction |

### Inventory Commands

| Class | Purpose |
|-------|---------|
| `GiveCommand` | Give items |
| `InventoryCommand` | Manage inventory |
| `InventoryClearCommand` | Clear inventory |
| `InventoryBackpackCommand` | Backpack management |
| `InventorySeeCommand` | View other inventories |

### Inventory Events

```java
// Entity inventory changes
LivingEntityInventoryChangeEvent

// Slot changes
SwitchActiveSlotEvent
```

---

## Item Categories

### FieldcraftCategory

**Class:** `FieldcraftCategory` - Field crafting categories

### ResourceType

**Class:** `ResourceType` - Resource categorization

---

## Related Files

- [07-blocks.md](07-blocks.md) - Block types
- [05-events.md](05-events.md) - Crafting events
