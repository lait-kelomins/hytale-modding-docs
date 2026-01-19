# Hytale ItemCategory System

## Overview

ItemCategories use a hierarchical parent-child system with dot notation (e.g., `Blocks.Rocks`).

## File Locations

| Type | Path |
|------|------|
| Creative Library Categories | `Server/Item/Category/CreativeLibrary/` |
| Fieldcraft Categories | `Server/Item/Category/Fieldcraft/` |
| Item Definitions | `Server/Item/Items/` |

## Category Structure

Parent categories are JSON files containing a `Children` array:

**Example:** `Server/Item/Category/CreativeLibrary/Blocks.json`
```json
{
  "Icon": "Icons/ItemCategories/Natural.png",
  "Order": 0,
  "Children": [
    { "Id": "Rocks", "Name": "server.ui.itemcategory.rocks", "Icon": "..." },
    { "Id": "Structural", "Name": "server.ui.itemcategory.structural", "Icon": "..." },
    { "Id": "Soils", "Name": "server.ui.itemcategory.soils", "Icon": "..." }
  ]
}
```

## Adding a Subcategory

To add a new subcategory (e.g., `Blocks.Fossils`):

1. Copy the existing parent category file (e.g., `Blocks.json`)
2. Add your new child to the `Children` array:
```json
{
  "Id": "Fossils",
  "Name": "server.ui.itemcategory.fossils",
  "Icon": "Icons/ItemCategories/Fossils.png"
}
```
3. Include all original children - the file replaces, not merges

**Important:** There is no patch/append mechanism. You must provide the complete file with all existing children plus your additions.

## Referencing Categories in Items

Items use dot notation in their `Categories` array:

```json
{
  "DisplayName": "Fossil Rock",
  "Categories": ["Blocks.Fossils"],
  "MaxStackSize": 64
}
```

Items can belong to multiple categories:
```json
{
  "Categories": ["Blocks.Rocks", "Items.Ingredients"]
}
```

## Known Categories

### Blocks
- `Blocks.Rocks`
- `Blocks.Structural`
- `Blocks.Soils`
- `Blocks.Ores`
- `Blocks.Plants`
- `Blocks.Fluids`
- `Blocks.Portals`
- `Blocks.Wood`
- `Blocks.Deco`

### Furniture
- `Furniture.Beds`
- `Furniture.Benches`
- `Furniture.Containers`
- `Furniture.Doors`
- `Furniture.Furniture`
- `Furniture.Lighting`
- `Furniture.Shelves`
- `Furniture.Signs`

### Items
- `Items.Armors`
- `Items.CombatMilestone2`
- `Items.Consumables`
- `Items.Debug`
- `Items.Foods`
- `Items.Ingredients`
- `Items.Potions`
- `Items.Recipes`
- `Items.Tools`
- `Items.Utility`
- `Items.Weapons`

### Tool (Creative Library)
- `Tool.BuilderTool`
- `Tool.BuilderToolSecondPage`
- `Tool.ScriptedBrushes`
- `Tool.Block`
- `Tool.BrushFilters`
- `Tool.Machinima`

## Java Classes

| Class | Path |
|-------|------|
| ItemCategory | `com.hypixel.hytale.server.core.asset.type.item.config.ItemCategory` |
| Protocol ItemCategory | `com.hypixel.hytale.protocol.ItemCategory` |
| Update Packet | `com.hypixel.hytale.protocol.packets.assets.UpdateItemCategories` |
| Packet Generator | `com.hypixel.hytale.server.core.asset.type.item.ItemCategoryPacketGenerator` |
| AssetStore | `com.hypixel.hytale.assetstore.AssetStore` |

## Runtime Category Registration (Advanced)

Categories CAN be added at runtime using reflection to access the AssetStore:

```java
// 1. Get classes via reflection
Class<?> itemCategoryClass = Class.forName(
    "com.hypixel.hytale.server.core.asset.type.item.config.ItemCategory");
Class<?> infoDisplayModeClass = Class.forName(
    "com.hypixel.hytale.protocol.ItemGridInfoDisplayMode");

// 2. Get display mode enum value
Object displayMode = Arrays.stream(infoDisplayModeClass.getEnumConstants())
    .filter(e -> e.toString().equals("Default"))
    .findFirst().orElse(null);

// 3. Create category using constructor(id, name, icon, displayMode, children[])
Constructor<?> ctor = itemCategoryClass.getConstructor(
    String.class, String.class, String.class,
    infoDisplayModeClass,
    java.lang.reflect.Array.newInstance(itemCategoryClass, 0).getClass());

Object newCategory = ctor.newInstance(
    "Blocks.Custom",                              // id (dot notation)
    "server.ui.itemcategory.custom",              // localization key
    "Icons/ItemCategories/Custom.png",            // icon path
    displayMode,                                  // ItemGridInfoDisplayMode
    java.lang.reflect.Array.newInstance(itemCategoryClass, 0)  // empty children
);

// 4. Register via AssetStore.loadAssets()
Method getAssetStore = itemCategoryClass.getMethod("getAssetStore");
Object assetStore = getAssetStore.invoke(null);

Method loadAssets = assetStore.getClass().getMethod("loadAssets",
    String.class, java.util.List.class);
loadAssets.invoke(assetStore, "mymod", java.util.List.of(newCategory));
```

### Key Methods

| Method | Description |
|--------|-------------|
| `ItemCategory.getAssetStore()` | Returns the AssetStore for categories |
| `ItemCategory.getAssetMap()` | Returns the DefaultAssetMap |
| `AssetStore.loadAssets(packName, List<T>)` | Registers new assets and syncs to clients |
| `DefaultAssetMap.getAssetMap()` | Returns internal Map for direct access |

### Notes

- The `loadAssets()` method triggers client sync via `UpdateItemCategories` packet
- Use a unique pack name (e.g., your mod name) to avoid conflicts
- Categories added this way persist until server restart
- For permanent categories, use asset files instead
