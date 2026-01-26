
This document provides a comprehensive reference of all asset types in the Hytale Server. Assets define game content such as blocks, items, entities, and more.

## Overview

Assets are JSON-defined game content loaded by the [Asset Store](Asset-Store) system. Each asset type has a specific schema and purpose.

## Asset Type Categories

- [Block Assets](#block-assets)
- [Item Assets](#item-assets)
- [Entity Assets](#entity-assets)
- [Audio Assets](#audio-assets)
- [Visual Assets](#visual-assets)
- [Environment Assets](#environment-assets)
- [UI Assets](#ui-assets)
- [Crafting Assets](#crafting-assets)
- [Combat Assets](#combat-assets)
- [Miscellaneous Assets](#miscellaneous-assets)

---

## Block Assets

### BlockType
Defines a block's properties and behavior.

**Location:** `com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockType`

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Block identifier |
| `displayName` | String | Localized display name |
| `solid` | Boolean | Whether the block is solid |
| `transparent` | Boolean | Whether light passes through |
| `hardness` | Float | Mining difficulty |
| `blastResistance` | Float | Explosion resistance |
| `flammable` | Boolean | Can catch fire |
| `replaceable` | Boolean | Can be replaced by placement |
| `collision` | CollisionType | Collision behavior |
| `lightLevel` | Integer | Light emission (0-15) |
| `lightAbsorption` | Integer | Light blocking (0-15) |
| `blockSet` | Reference | Visual block set |
| `soundSet` | Reference | Sound effects |
| `particleSet` | Reference | Particle effects |
| `drops` | DropTable | Block drops |
| `tickProcedure` | Reference | Tick behavior |
| `placementSettings` | Object | Placement rules |
| `gathering` | Object | Tool requirements |

**Example:**
```json
{
  "name": "stone",
  "displayName": "Stone",
  "solid": true,
  "transparent": false,
  "hardness": 1.5,
  "blastResistance": 6.0,
  "lightLevel": 0,
  "blockSet": "stone_set",
  "soundSet": "stone_sounds",
  "gathering": {
    "tool": "pickaxe",
    "minTier": 1
  },
  "drops": [
    { "item": "cobblestone", "count": 1 }
  ]
}
```

### BlockSet
Visual appearance configuration.

| Property | Type | Description |
|----------|------|-------------|
| `model` | Reference | 3D model |
| `textures` | Map | Face textures |
| `variants` | List | Visual variants |

### BlockSoundSet
Sound effects for blocks.

| Property | Type | Description |
|----------|------|-------------|
| `place` | Sound | Placement sound |
| `break` | Sound | Breaking sound |
| `step` | Sound | Walking sound |
| `hit` | Sound | Hit sound |

### BlockParticleSet
Particle effects for blocks.

| Property | Type | Description |
|----------|------|-------------|
| `break` | Particle | Break particles |
| `step` | Particle | Walk particles |

### BlockBoundingBoxes
Collision and selection boxes.

| Property | Type | Description |
|----------|------|-------------|
| `collision` | Box[] | Collision boxes |
| `selection` | Box[] | Selection boxes |
| `rotatedVariants` | Map | Per-rotation boxes |

### BlockBreakingDecal
Breaking animation textures.

---

## Item Assets

### ItemType
Defines an item's properties.

**Location:** `com.hypixel.hytale.server.core.asset.type.itemtype.config.ItemType`

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Item identifier |
| `displayName` | String | Display name |
| `stackSize` | Integer | Max stack size |
| `rarity` | Rarity | Item rarity |
| `durability` | Integer | Max durability |
| `model` | Reference | 3D model |
| `icon` | Reference | Inventory icon |
| `category` | Category | Item category |
| `attributes` | Map | Stat modifiers |
| `useAction` | Action | Right-click action |

**Example:**
```json
{
  "name": "iron_sword",
  "displayName": "Iron Sword",
  "stackSize": 1,
  "rarity": "common",
  "durability": 250,
  "category": "weapon",
  "attributes": {
    "attackDamage": 6.0,
    "attackSpeed": 1.6
  }
}
```

### WeaponType
Extended item type for weapons.

| Property | Type | Description |
|----------|------|-------------|
| `attackDamage` | Float | Base damage |
| `attackSpeed` | Float | Attack speed |
| `reach` | Float | Attack range |
| `knockback` | Float | Knockback strength |
| `sweeping` | Boolean | Has sweep attack |
| `combos` | ComboData | Combo attacks |

### ToolType
Tools for gathering and building.

| Property | Type | Description |
|----------|------|-------------|
| `toolType` | Enum | pickaxe/axe/shovel/hoe |
| `tier` | Integer | Tool tier |
| `efficiency` | Float | Mining speed |
| `enchantability` | Integer | Enchant modifier |

### ArmorType
Protective equipment.

| Property | Type | Description |
|----------|------|-------------|
| `slot` | Enum | head/chest/legs/feet |
| `defense` | Float | Damage reduction |
| `toughness` | Float | Heavy hit protection |
| `weight` | Float | Movement penalty |

---

## Entity Assets

### EntityType
Defines entity templates.

**Location:** `com.hypixel.hytale.server.core.asset.type.entitytype.config.EntityType`

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Entity identifier |
| `displayName` | String | Display name |
| `health` | Float | Max health |
| `speed` | Float | Movement speed |
| `model` | Reference | 3D model |
| `hitbox` | Box | Collision box |
| `components` | List | Default components |
| `behaviors` | List | AI behaviors |
| `drops` | DropTable | Death drops |
| `attributes` | Map | Base attributes |

**Example:**
```json
{
  "name": "zombie",
  "displayName": "Zombie",
  "health": 20.0,
  "speed": 0.23,
  "hitbox": { "width": 0.6, "height": 1.95 },
  "behaviors": [
    { "type": "wander" },
    { "type": "attackMelee", "damage": 3.0 }
  ],
  "drops": [
    { "item": "rotten_flesh", "count": "0-2" }
  ]
}
```

### EntityEffect
Status effects for entities.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Effect identifier |
| `icon` | Reference | UI icon |
| `color` | Color | Particle color |
| `positive` | Boolean | Beneficial effect |
| `maxLevel` | Integer | Max strength |
| `modifiers` | List | Stat changes |

### EntityUIComponent
HUD elements for entities.

---

## Audio Assets

### AudioCategory
Audio channel configuration.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Category name |
| `defaultVolume` | Float | Default volume |
| `parent` | Reference | Parent category |

### SoundEvent
Individual sound definitions.

| Property | Type | Description |
|----------|------|-------------|
| `sounds` | List | Sound file variants |
| `volume` | Range | Volume range |
| `pitch` | Range | Pitch range |
| `category` | Reference | Audio category |

---

## Visual Assets

### Model
3D model definition.

| Property | Type | Description |
|----------|------|-------------|
| `geometry` | File | Model file (.hym) |
| `textures` | Map | Texture bindings |
| `animations` | Map | Animation bindings |

### Animation
Animation definition.

| Property | Type | Description |
|----------|------|-------------|
| `file` | File | Animation file (.hya) |
| `loop` | Boolean | Loop animation |
| `speed` | Float | Playback speed |

### Particle
Particle system definition.

| Property | Type | Description |
|----------|------|-------------|
| `texture` | Reference | Particle texture |
| `count` | Range | Particle count |
| `lifetime` | Range | Particle lifetime |
| `speed` | Range | Movement speed |
| `gravity` | Float | Gravity effect |
| `color` | ColorRange | Color over lifetime |

---

## Environment Assets

### Environment
Biome/environment definition.

**Location:** `com.hypixel.hytale.server.core.asset.type.environment.config.Environment`

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Environment ID |
| `displayName` | String | Display name |
| `temperature` | Float | Temperature value |
| `humidity` | Float | Humidity value |
| `skyColor` | Color | Sky color |
| `fogColor` | Color | Fog color |
| `waterColor` | Color | Water tint |
| `ambience` | Reference | Ambient sounds |
| `music` | Reference | Background music |
| `surfaceBlock` | Reference | Surface block |
| `fillerBlock` | Reference | Filler block |

### AmbienceFX
Ambient sound/visual effects.

| Property | Type | Description |
|----------|------|-------------|
| `ambientBed` | Reference | Ambient sound loop |
| `soundEffects` | List | Random sounds |
| `music` | Reference | Music tracks |
| `conditions` | Object | Play conditions |

### Weather
Weather type definition.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Weather ID |
| `particles` | Reference | Weather particles |
| `skyModifier` | Color | Sky tint |
| `fogModifier` | Float | Fog density |
| `lightModifier` | Float | Light level change |

---

## UI Assets

### CustomUI
Custom UI layout definition.

| Property | Type | Description |
|----------|------|-------------|
| `elements` | List | UI elements |
| `bindings` | Map | Data bindings |
| `events` | Map | Event handlers |

### Objective
Quest/objective definition.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Objective ID |
| `displayName` | String | Display text |
| `description` | String | Description |
| `icon` | Reference | UI icon |
| `requirements` | List | Completion requirements |
| `rewards` | List | Completion rewards |

---

## Crafting Assets

### Recipe
Crafting recipe definition.

| Property | Type | Description |
|----------|------|-------------|
| `type` | Enum | shaped/shapeless/processing |
| `ingredients` | List | Input items |
| `result` | Object | Output item |
| `bench` | Reference | Required bench |
| `duration` | Float | Crafting time |

**Example (Shaped):**
```json
{
  "type": "shaped",
  "pattern": [
    " I ",
    " I ",
    " S "
  ],
  "key": {
    "I": "iron_ingot",
    "S": "stick"
  },
  "result": {
    "item": "iron_sword",
    "count": 1
  }
}
```

### CraftingBench
Crafting station definition.

| Property | Type | Description |
|----------|------|-------------|
| `categories` | List | Recipe categories |
| `slots` | Object | Input/output slots |
| `tiers` | List | Upgrade tiers |

### ProcessingBench
Processing station (furnace, etc.).

| Property | Type | Description |
|----------|------|-------------|
| `processingTime` | Float | Base process time |
| `fuelSlot` | Boolean | Has fuel slot |
| `extraOutputs` | List | Bonus outputs |

---

## Combat Assets

### DamageType
Damage category definition.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Damage type ID |
| `bypassesArmor` | Boolean | Ignores defense |
| `magical` | Boolean | Magical damage |
| `deathMessage` | String | Death message key |

### Projectile
Projectile definition.

| Property | Type | Description |
|----------|------|-------------|
| `speed` | Float | Movement speed |
| `gravity` | Float | Gravity effect |
| `damage` | Float | Impact damage |
| `piercing` | Integer | Entities pierced |
| `model` | Reference | Projectile model |

### Explosion
Explosion configuration.

| Property | Type | Description |
|----------|------|-------------|
| `radius` | Float | Explosion radius |
| `damage` | Float | Max damage |
| `blockDamage` | Boolean | Destroys blocks |
| `fire` | Boolean | Creates fire |

---

## Miscellaneous Assets

### LootTable
Drop/loot configuration.

| Property | Type | Description |
|----------|------|-------------|
| `pools` | List | Loot pools |
| `conditions` | List | Drop conditions |

**Example:**
```json
{
  "pools": [
    {
      "rolls": 1,
      "entries": [
        { "item": "diamond", "weight": 1 },
        { "item": "emerald", "weight": 2 }
      ],
      "conditions": [
        { "type": "randomChance", "chance": 0.1 }
      ]
    }
  ]
}
```

### TagSet
Entity/block tag grouping.

| Property | Type | Description |
|----------|------|-------------|
| `entries` | List | Tagged items |

### Attitude
NPC attitude configuration.

| Property | Type | Description |
|----------|------|-------------|
| `name` | String | Attitude ID |
| `hostile` | Boolean | Attacks players |
| `fearful` | Boolean | Flees from players |

---

## Asset File Locations

Assets are typically organized:

```
assets/
├── blocks/
│   ├── stone.json
│   └── dirt.json
├── items/
│   ├── weapons/
│   └── tools/
├── entities/
│   ├── hostile/
│   └── passive/
├── environments/
├── recipes/
└── sounds/
```

## See Also

- [Asset Store](Asset-Store) - Asset loading system
- [Block Types](Block-Types) - Detailed block documentation
- [Entity Types](Entity-Types) - Entity configuration details
- [Creating Plugins](Creating-Plugins) - Custom asset registration
