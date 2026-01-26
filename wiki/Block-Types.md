
This document provides detailed documentation on block type configuration and the block system in the Hytale Server.

## Overview

Blocks are the fundamental building units of the game world. Each block type defines visual appearance, physical properties, behavior, and interactions.

## Block Type Structure

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockType`

```
BlockType
├── Basic Properties (name, solid, transparent)
├── Physics Properties (hardness, resistance)
├── Visual Configuration (blockSet, lightLevel)
├── Audio Configuration (soundSet)
├── Interaction Configuration (gathering, placement)
├── State Configuration (states, tickProcedure)
└── Drop Configuration (drops)
```

## Basic Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | String | Required | Unique block identifier |
| `displayName` | String | name | Localized display name |
| `solid` | Boolean | true | Blocks entity movement |
| `transparent` | Boolean | false | Light passes through |
| `replaceable` | Boolean | false | Can be replaced by placing |
| `fullCube` | Boolean | true | Full 1x1x1 block |

## Physics Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `hardness` | Float | 1.0 | Mining time multiplier |
| `blastResistance` | Float | 0.0 | Explosion resistance |
| `flammable` | Boolean | false | Can catch/spread fire |
| `fireSpreadSpeed` | Integer | 0 | Fire spread rate |
| `burnChance` | Integer | 0 | Chance to burn away |
| `friction` | Float | 0.6 | Surface friction |
| `slipperiness` | Float | 0.6 | Ice-like sliding |

## Visual Properties

| Property | Type | Description |
|----------|------|-------------|
| `blockSet` | Reference | Visual appearance (model, textures) |
| `lightLevel` | Integer (0-15) | Light emission |
| `lightAbsorption` | Integer (0-15) | Light blocking |
| `renderLayer` | Enum | solid/cutout/translucent |
| `ambientOcclusion` | Boolean | Enable AO shading |

## Audio Properties

| Property | Type | Description |
|----------|------|-------------|
| `soundSet` | Reference | Sound effects |
| `particleSet` | Reference | Particle effects |

## Block Faces

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockFace`

Defines per-face properties:

```java
public enum BlockFace {
    UP,
    DOWN,
    NORTH,
    SOUTH,
    EAST,
    WEST
}
```

### Face Connection Types

```java
public enum FaceConnectionType {
    NONE,       // No connection
    FULL,       // Full face connection
    PARTIAL,    // Partial connection
    OVERLAY     // Overlay texture
}
```

### Face Support

**Location:** `BlockFaceSupport`

| Property | Type | Description |
|----------|------|-------------|
| `providesSupport` | FaceSet | Faces that support blocks |
| `requiresSupport` | FaceSet | Faces that need support |
| `supportType` | Enum | full/partial/center |

## Placement Settings

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockPlacementSettings`

| Property | Type | Description |
|----------|------|-------------|
| `rotationMode` | RotationMode | How block rotates on place |
| `canPlaceOn` | BlockFilter | Valid support blocks |
| `canPlaceAt` | PositionFilter | Valid positions |
| `previewVisibility` | Enum | Preview display mode |

### Rotation Modes

```java
public enum RotationMode {
    NONE,           // No rotation
    HORIZONTAL,     // 4 horizontal rotations
    ALL,            // 24 rotations
    PLAYER_FACING,  // Face toward player
    CLICK_FACE      // Align to clicked face
}
```

### Preview Visibility

```java
public enum BlockPreviewVisibility {
    ALWAYS,
    NEVER,
    WHEN_VALID,
    WHEN_INVALID
}
```

## Block Movement Settings

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockMovementSettings`

| Property | Type | Description |
|----------|------|-------------|
| `walkSpeed` | Float | Walk speed modifier |
| `jumpModifier` | Float | Jump height modifier |
| `climbable` | Boolean | Can climb like ladder |
| `swimmable` | Boolean | Swimming behavior |
| `bouncy` | Float | Bounce factor |

## Gathering (Mining)

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockGathering`

| Property | Type | Description |
|----------|------|-------------|
| `tool` | ToolType | Required tool type |
| `minTier` | Integer | Minimum tool tier |
| `efficiency` | Float | Base mining speed |
| `silkTouchable` | Boolean | Can silk touch |

### Block Tool Data

```java
public class BlockToolData {
    ToolType toolType;      // pickaxe, axe, shovel, etc.
    int tierRequired;       // 0=hand, 1=wood, 2=stone, etc.
    float speedMultiplier;  // Tool efficiency bonus
    boolean requiresTool;   // Must use tool to drop
}
```

## Block Drops

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockBreakingDropType`

| Property | Type | Description |
|----------|------|-------------|
| `item` | Reference | Dropped item |
| `count` | Range | Drop count |
| `chance` | Float | Drop probability |
| `conditions` | List | Drop conditions |
| `fortuneBonus` | Float | Fortune modifier |

### Drop Types

```java
public enum BlockBreakingDropType {
    ALWAYS,         // Always drops
    SILK_TOUCH,     // Only with silk touch
    NO_SILK_TOUCH,  // Only without silk touch
    FORTUNE         // Affected by fortune
}
```

## Block States

Blocks can have multiple states with different properties.

### State Properties

| Property | Type | Description |
|----------|------|-------------|
| `facing` | Direction | Block orientation |
| `powered` | Boolean | Redstone powered |
| `waterlogged` | Boolean | Contains water |
| `age` | Integer | Growth stage |
| `level` | Integer | Fluid level |

### State Registry

```java
// Register block state
BlockStateRegistry registry = plugin.getBlockStateRegistry();
registry.register(MyBlockState.class, MyBlockState.CODEC);
```

### Tickable Block State

```java
public interface TickableBlockState {
    void onTick(World world, int x, int y, int z);
}
```

## Block Tick Procedure

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktick.config.TickProcedure`

| Property | Type | Description |
|----------|------|-------------|
| `tickRate` | Integer | Ticks between updates |
| `randomTick` | Boolean | Random tick enabled |
| `scheduledTick` | Boolean | Scheduled tick enabled |

## Mount Points

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.mountpoints`

For blocks that entities can sit on or attach to:

| Property | Type | Description |
|----------|------|-------------|
| `position` | Vector3f | Mount position |
| `rotation` | Vector3f | Mount rotation |
| `playerOffset` | Vector3f | Player offset |

## Crafting Benches

### CraftingBench

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.bench.CraftingBench`

| Property | Type | Description |
|----------|------|-------------|
| `categories` | List | Recipe categories |
| `slots` | BenchSlot[] | Crafting slots |
| `gridSize` | Vector2i | Crafting grid |

### ProcessingBench

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.bench.ProcessingBench`

| Property | Type | Description |
|----------|------|-------------|
| `processingTime` | Float | Base process time |
| `fuelSlot` | Boolean | Has fuel slot |
| `inputSlots` | BenchSlot[] | Input slots |
| `outputSlots` | BenchSlot[] | Output slots |
| `extraOutputs` | List | Bonus outputs |

### DiagramCraftingBench

For pattern-based crafting.

### StructuralCraftingBench

For multi-block structures.

### Bench Tiers

```java
public class BenchTierLevel {
    int tier;
    List<String> unlockedRecipes;
    BenchUpgradeRequirement upgradeRequirement;
}
```

## Farming Data

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.farming`

| Property | Type | Description |
|----------|------|-------------|
| `stages` | List | Growth stages |
| `growthTime` | Range | Time per stage |
| `soilConfig` | SoilConfig | Soil requirements |
| `modifiers` | List | Growth modifiers |

### Farming Stage Data

```java
public class FarmingStageData {
    BlockType blockType;      // Block for this stage
    int minAge, maxAge;       // Age range
    float growthChance;       // Growth probability
}
```

### Growth Modifiers

- `FertilizerGrowthModifierAsset` - Fertilizer bonus
- `LightLevelGrowthModifierAsset` - Light requirements
- `WaterGrowthModifierAsset` - Water proximity

## Block Migration

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockMigration`

For converting old block types:

| Property | Type | Description |
|----------|------|-------------|
| `oldId` | String | Previous block ID |
| `newId` | String | New block ID |
| `stateMapping` | Map | State conversion |

## Example Block Definitions

### Simple Solid Block

```json
{
    "name": "stone",
    "displayName": "Stone",
    "solid": true,
    "transparent": false,
    "hardness": 1.5,
    "blastResistance": 6.0,
    "blockSet": "stone_set",
    "soundSet": "stone_sounds",
    "particleSet": "stone_particles",
    "gathering": {
        "tool": "pickaxe",
        "minTier": 1,
        "requiresTool": true
    },
    "drops": [
        {
            "type": "NO_SILK_TOUCH",
            "item": "cobblestone",
            "count": 1
        },
        {
            "type": "SILK_TOUCH",
            "item": "stone",
            "count": 1
        }
    ]
}
```

### Transparent Block (Glass)

```json
{
    "name": "glass",
    "displayName": "Glass",
    "solid": true,
    "transparent": true,
    "fullCube": false,
    "hardness": 0.3,
    "blockSet": "glass_set",
    "soundSet": "glass_sounds",
    "renderLayer": "cutout",
    "ambientOcclusion": false,
    "gathering": {
        "silkTouchable": true
    },
    "drops": []
}
```

### Crop Block

```json
{
    "name": "wheat",
    "displayName": "Wheat",
    "solid": false,
    "transparent": true,
    "hardness": 0.0,
    "replaceable": true,
    "blockSet": "wheat_set",
    "placement": {
        "canPlaceOn": ["farmland"],
        "requiresSupport": ["DOWN"]
    },
    "farming": {
        "stages": [
            { "age": 0, "block": "wheat_stage_0" },
            { "age": 7, "block": "wheat_stage_7" }
        ],
        "growthTime": { "min": 1200, "max": 2400 },
        "modifiers": [
            { "type": "light", "minLight": 9 }
        ]
    }
}
```

### Crafting Table

```json
{
    "name": "crafting_table",
    "displayName": "Crafting Table",
    "solid": true,
    "hardness": 2.5,
    "blockSet": "crafting_table_set",
    "soundSet": "wood_sounds",
    "gathering": {
        "tool": "axe"
    },
    "bench": {
        "type": "crafting",
        "gridSize": [3, 3],
        "categories": ["all"]
    }
}
```

## Block Rotation Utility

**Location:** `com.hypixel.hytale.server.core.universe.world.chunk.BlockRotationUtil`

Utilities for block rotation calculations:

```java
BlockRotationUtil
├── getRotatedFace(face, rotation)
├── getRotationFromFacing(direction)
├── applyRotation(blockState, rotation)
└── getFlippedRotation(rotation, flipType)
```

### Flip Types

```java
public enum BlockFlipType {
    NONE,
    X_AXIS,
    Z_AXIS,
    BOTH
}
```

## Block Supports Required

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockSupportsRequiredForType`

Defines support requirements:

| Property | Type | Description |
|----------|------|-------------|
| `requiresFullSupport` | Boolean | Needs full block below |
| `requiresCenterSupport` | Boolean | Needs center support |
| `supportFaces` | FaceSet | Faces providing support |

## See Also

- [Asset Types](Asset-Types) - All asset types reference
- [World Universe](World-Universe) - Chunk and block storage
- [Asset Store](Asset-Store) - Asset loading system
