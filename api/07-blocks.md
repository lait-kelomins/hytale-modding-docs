# Hytale Blocks

> **Index:** [Block Types](#block-types) | [Block Events](#block-events) | [Block Properties](#block-properties)

---

## Block System

**Key Packages:**
```
com.hypixel.hytale.server.core.asset.type.blocktype/
com.hypixel.hytale.server.core.asset.type.blocktype.config/
com.hypixel.hytale.server.core.blocktype/
```

---

## Block Types

**Main Class:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockType`

### Block Properties

| Class | Purpose |
|-------|---------|
| `BlockFace` | Block face data |
| `BlockFaceSupport` | Face support requirements |
| `BlockFlipType` | Flip/rotation types |
| `BlockGathering` | Harvesting/gathering data |
| `BlockMigration` | Version migration |
| `BlockMovementSettings` | Movement effects |
| `BlockPlacementSettings` | Placement rules |
| `BlockTypeTextures` | Texture definitions |

---

## Block Events

**Package:** `com.hypixel.hytale.server.core.event.events.ecs`

### Break Block

```java
// Event: BreakBlockEvent
public class MyBlockBreakHandler extends EntityEventSystem<EntityStore, BreakBlockEvent> {
    public MyBlockBreakHandler() {
        super(BreakBlockEvent.class);
    }

    @Override
    public void handle(..., BreakBlockEvent event) {
        // Handle block break
    }
}
```

### Place Block

```java
// Event: PlaceBlockEvent
```

### Use/Interact Block

```java
// Event: UseBlockEvent
// Event: UseBlockEvent.Pre  (before interaction)
// Event: UseBlockEvent.Post (after interaction)
```

### Damage Block

```java
// Event: DamageBlockEvent
```

---

## Block Placement

**Class:** `BlockPlacementSettings`

Properties:
- `BlockPreviewVisibility` - Preview when placing
- `RotationMode` - How block rotates

---

## Block Support

**Class:** `RequiredBlockFaceSupport`

Defines what faces need support for the block to remain placed.

---

## Block Gathering/Breaking

**Class:** `BlockGathering`

- `BlockToolData` - Tool requirements for breaking

### Drop Types

| Class | Purpose |
|-------|---------|
| `BlockBreakingDropType` | Drops when broken |
| `HarvestingDropType` | Drops when harvested |
| `PhysicsDropType` | Physics-based drops |
| `SoftBlockDropType` | Soft block drops |
| `SupportDropType` | Support-loss drops |

---

## Block Rotation

**Classes:**
- `Rotation` - Rotation states
- `RotationTuple` - Rotation data
- `VariantRotation` - Variant-specific rotation

---

## Block Hitbox/Collision

**Package:** `com.hypixel.hytale.server.core.asset.type.blockhitbox`

| Class | Purpose |
|-------|---------|
| `BlockBoundingBoxes` | Collision boxes |
| `RotatedVariantBoxes` | Per-rotation boxes |

---

## Block Sounds

**Package:** `com.hypixel.hytale.server.core.asset.type.blocksound`

**Class:** `BlockSoundSet` - Sound effects for blocks

---

## Block Particles

**Package:** `com.hypixel.hytale.server.core.asset.type.blockparticle`

**Class:** `BlockParticleSet` - Particle effects

---

## Farming/Growth

**Package:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.farming`

| Class | Purpose |
|-------|---------|
| `FarmingData` | Crop farming data |
| `FarmingStageData` | Growth stages |
| `GrowthModifierAsset` | Growth modifiers |
| `SoilConfig` | Soil requirements |

---

## Block Ticking

**Package:** `com.hypixel.hytale.server.core.asset.type.blocktick`

| Class | Purpose |
|-------|---------|
| `BlockTickManager` | Tick management |
| `BlockTickStrategy` | Tick behavior |
| `TickProcedure` | Tick procedures |
| `IBlockTickProvider` | Tick provider interface |

---

## Bench Blocks

Crafting stations and workbenches.

**Package:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.bench`

| Class | Purpose |
|-------|---------|
| `Bench` | Base bench block |
| `CraftingBench` | Crafting table |
| `ProcessingBench` | Furnace-like |
| `DiagramCraftingBench` | Pattern crafting |
| `StructuralCraftingBench` | Building |
| `BenchTierLevel` | Upgrade levels |
| `BenchUpgradeRequirement` | Upgrade requirements |

---

## Fluids

**Package:** `com.hypixel.hytale.server.core.asset.type.fluid`

| Class | Purpose |
|-------|---------|
| `Fluid` | Fluid definition |
| `FluidTicker` | Fluid simulation |
| `FiniteFluidTicker` | Finite fluid sim |
| `DefaultFluidTicker` | Default behavior |

---

## Related Files

- [02-ecs-system.md](02-ecs-system.md) - Block event handling
- [06-items.md](06-items.md) - Item drops
