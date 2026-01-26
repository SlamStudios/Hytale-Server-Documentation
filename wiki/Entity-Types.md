
This document provides detailed documentation on entity type configuration and the entity system in the Hytale Server.

## Overview

Entities are dynamic game objects including players, NPCs, creatures, items, and projectiles. Each entity type defines appearance, behavior, stats, and interactions.

## Entity Type Structure

**Location:** `com.hypixel.hytale.server.core.asset.type.entitytype.config.EntityType`

```
EntityType
├── Identity (name, displayName)
├── Stats (health, speed, damage)
├── Visual (model, animations)
├── Physics (hitbox, collision)
├── Components (default components)
├── Behaviors (AI behaviors)
├── Drops (loot table)
└── Attributes (base stats)
```

## Basic Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | String | Required | Unique entity identifier |
| `displayName` | String | name | Localized display name |
| `category` | EntityCategory | MISC | Entity category |
| `persistent` | Boolean | true | Survives chunk unload |
| `summonable` | Boolean | true | Can be summoned |

## Entity Categories

```java
public enum EntityCategory {
    PLAYER,         // Player entities
    HOSTILE,        // Hostile mobs
    PASSIVE,        // Passive mobs
    AMBIENT,        // Ambient creatures
    WATER_CREATURE, // Water mobs
    NPC,            // Non-player characters
    PROJECTILE,     // Projectiles
    ITEM,           // Dropped items
    VEHICLE,        // Rideable vehicles
    BLOCK_ENTITY,   // Block-attached entities
    MISC            // Miscellaneous
}
```

## Stats Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `health` | Float | 20.0 | Maximum health |
| `armor` | Float | 0.0 | Damage reduction |
| `attackDamage` | Float | 1.0 | Base attack damage |
| `movementSpeed` | Float | 0.3 | Movement speed |
| `knockbackResistance` | Float | 0.0 | Knockback reduction |
| `followRange` | Float | 16.0 | Target detection range |
| `attackKnockback` | Float | 0.4 | Attack knockback strength |

## Visual Properties

| Property | Type | Description |
|----------|------|-------------|
| `model` | Reference | 3D model file |
| `texture` | Reference | Entity texture |
| `animations` | Map | Animation bindings |
| `scale` | Float | Model scale |
| `shadowRadius` | Float | Shadow size |

### Animation Bindings

```json
{
    "animations": {
        "idle": "entity/zombie/idle.hya",
        "walk": "entity/zombie/walk.hya",
        "run": "entity/zombie/run.hya",
        "attack": "entity/zombie/attack.hya",
        "death": "entity/zombie/death.hya"
    }
}
```

## Physics Properties

| Property | Type | Description |
|----------|------|-------------|
| `hitbox` | Box | Collision/hitbox size |
| `eyeHeight` | Float | Eye position height |
| `stepHeight` | Float | Max step height |
| `gravity` | Float | Gravity multiplier |
| `pushable` | Boolean | Can be pushed |
| `collidable` | Boolean | Has collision |

### Hitbox Definition

```json
{
    "hitbox": {
        "width": 0.6,
        "height": 1.95,
        "depth": 0.6
    }
}
```

## Entity Attributes

**Location:** `com.hypixel.hytale.server.core.entity.StatModifiersManager`

Base attributes that can be modified:

| Attribute | Description |
|-----------|-------------|
| `maxHealth` | Maximum health points |
| `movementSpeed` | Movement velocity |
| `attackDamage` | Base attack damage |
| `attackSpeed` | Attacks per second |
| `armor` | Damage reduction |
| `armorToughness` | Heavy hit protection |
| `knockbackResistance` | Knockback reduction |
| `luck` | Loot luck modifier |

### Attribute Modifiers

```json
{
    "attributes": {
        "maxHealth": {
            "base": 20.0,
            "modifiers": [
                {
                    "name": "health_boost",
                    "operation": "ADD",
                    "amount": 10.0
                }
            ]
        }
    }
}
```

### Modifier Operations

```java
public enum ModifierOperation {
    ADD,            // base + amount
    MULTIPLY_BASE,  // base * (1 + amount)
    MULTIPLY_TOTAL  // total * (1 + amount)
}
```

## Components

Default components attached to entities:

### Core Components

| Component | Description |
|-----------|-------------|
| `UUIDComponent` | Unique identifier |
| `HealthComponent` | Health management |
| `DamageDataComponent` | Damage tracking |
| `MovementStatesComponent` | Movement state |
| `EffectControllerComponent` | Status effects |
| `KnockbackComponent` | Knockback handling |

### Player-Specific Components

| Component | Description |
|-----------|-------------|
| `PlayerConfigData` | Player configuration |
| `PlayerWorldData` | World-specific data |
| `PlayerDeathPositionData` | Death location |
| `PlayerRespawnPointData` | Spawn point |
| `UniqueItemUsagesComponent` | Item usage tracking |

### Component Configuration

```json
{
    "components": [
        {
            "type": "health",
            "maxHealth": 20.0,
            "regeneration": 0.5
        },
        {
            "type": "movement",
            "speed": 0.3,
            "jumpStrength": 0.42
        }
    ]
}
```

## AI Behaviors

**Location:** `com.hypixel.hytale.builtin.npc*`

### Behavior Types

| Behavior | Description |
|----------|-------------|
| `wander` | Random wandering |
| `lookAtPlayer` | Look at nearby players |
| `followOwner` | Follow owner (pets) |
| `attackMelee` | Melee attacks |
| `attackRanged` | Ranged attacks |
| `flee` | Flee from threats |
| `panic` | Panic when hurt |
| `swim` | Swimming behavior |
| `float` | Float in water |
| `avoidEntity` | Avoid certain entities |
| `tempt` | Follow item holders |
| `breed` | Breeding behavior |

### Behavior Configuration

```json
{
    "behaviors": [
        {
            "type": "wander",
            "priority": 5,
            "speed": 0.8,
            "radius": 10
        },
        {
            "type": "attackMelee",
            "priority": 2,
            "speed": 1.0,
            "damage": 3.0,
            "cooldown": 20
        },
        {
            "type": "lookAtPlayer",
            "priority": 8,
            "range": 8.0
        }
    ]
}
```

### Behavior Priority

Lower priority = higher importance. Behaviors execute in priority order until one succeeds.

## Target Selectors

| Selector | Description |
|----------|-------------|
| `nearestPlayer` | Closest player |
| `nearestHostile` | Closest hostile |
| `hurtBy` | Entity that attacked |
| `owner` | Pet owner |
| `randomTarget` | Random valid target |

### Target Configuration

```json
{
    "targeting": {
        "selector": "nearestPlayer",
        "range": 16.0,
        "mustSee": true,
        "mustReach": true,
        "conditions": [
            { "type": "notCreative" }
        ]
    }
}
```

## Drops / Loot

| Property | Type | Description |
|----------|------|-------------|
| `lootTable` | Reference | Loot table reference |
| `experience` | Range | XP dropped |

### Drop Configuration

```json
{
    "drops": {
        "lootTable": "entities/zombie",
        "experience": { "min": 5, "max": 8 }
    }
}
```

### Inline Drops

```json
{
    "drops": [
        {
            "item": "rotten_flesh",
            "count": { "min": 0, "max": 2 },
            "chance": 1.0
        },
        {
            "item": "iron_ingot",
            "count": 1,
            "chance": 0.025,
            "conditions": [
                { "type": "killedByPlayer" }
            ]
        }
    ]
}
```

## Entity Effects

**Location:** `com.hypixel.hytale.server.core.entity.effect`

### ActiveEntityEffect

```java
public class ActiveEntityEffect {
    EntityEffect effect;    // Effect type
    float duration;         // Remaining duration
    float strength;         // Effect level
    Entity source;          // Effect source
}
```

### Effect Controller

```java
EffectControllerComponent controller = entity.getComponent(EFFECT_CONTROLLER);
controller.addEffect(new ActiveEntityEffect(effect, duration, strength));
controller.removeEffect(effectType);
List<ActiveEntityEffect> active = controller.getActiveEffects();
```

## Entity Events

### Spawn/Despawn

| Event | Description |
|-------|-------------|
| `EntitySpawnEvent` | Entity spawned |
| `EntityDespawnEvent` | Entity despawned |
| `EntityLoadEvent` | Entity loaded from storage |
| `EntityUnloadEvent` | Entity unloaded |

### Damage Events

| Event | Description |
|-------|-------------|
| `EntityDamageEvent` | Entity took damage |
| `EntityDeathEvent` | Entity died |
| `EntityHealEvent` | Entity healed |

### Movement Events

| Event | Description |
|-------|-------------|
| `EntityMoveEvent` | Entity moved |
| `EntityTeleportEvent` | Entity teleported |

## Player Entity

**Location:** `com.hypixel.hytale.server.core.entity.entities.Player`

Special entity type for players:

### Player Managers

| Manager | Description |
|---------|-------------|
| `MovementManager` | Movement handling |
| `WindowManager` | UI windows |
| `PageManager` | UI pages |
| `HudManager` | HUD elements |
| `CameraManager` | Camera control |
| `HotbarManager` | Hotbar management |
| `HiddenPlayersManager` | Player visibility |

### Player Data

```java
Player player = ...;

// Identity
UUID uuid = player.getUuid();
String username = player.getUsername();

// Position
Vector3f position = player.getPosition();
World world = player.getWorld();

// Stats
float health = player.getHealth();
float maxHealth = player.getMaxHealth();

// Inventory
Inventory inventory = player.getInventory();
ItemStack heldItem = player.getHeldItem();

// Network
PacketHandler handler = player.getPacketHandler();
player.sendPacket(packet);
```

## Living Entity

**Location:** `com.hypixel.hytale.server.core.entity.LivingEntity`

Base class for entities with health:

```java
public class LivingEntity extends Entity {
    float getHealth();
    void setHealth(float health);
    float getMaxHealth();
    boolean isDead();

    void damage(float amount, DamageSource source);
    void heal(float amount);

    void addEffect(EntityEffect effect);
    void removeEffect(EntityEffect effect);
    boolean hasEffect(EntityEffect effect);
}
```

## Block Entity

**Location:** `com.hypixel.hytale.server.core.entity.entities.BlockEntity`

Entities attached to blocks:

```java
public class BlockEntity extends Entity {
    BlockPosition getBlockPosition();
    BlockType getBlockType();
}
```

## Projectile Component

**Location:** `com.hypixel.hytale.server.core.entity.entities.ProjectileComponent`

For projectile entities:

| Property | Type | Description |
|----------|------|-------------|
| `shooter` | Entity | Who fired |
| `damage` | Float | Impact damage |
| `piercing` | Integer | Entities pierced |
| `gravity` | Boolean | Affected by gravity |
| `inGround` | Boolean | Stuck in block |

## Example Entity Definitions

### Hostile Mob (Zombie)

```json
{
    "name": "zombie",
    "displayName": "Zombie",
    "category": "HOSTILE",

    "health": 20.0,
    "movementSpeed": 0.23,
    "attackDamage": 3.0,
    "followRange": 35.0,

    "model": "entities/zombie/zombie.hym",
    "texture": "entities/zombie/zombie.png",
    "animations": {
        "idle": "entities/zombie/idle.hya",
        "walk": "entities/zombie/walk.hya",
        "attack": "entities/zombie/attack.hya"
    },

    "hitbox": {
        "width": 0.6,
        "height": 1.95
    },

    "behaviors": [
        { "type": "float", "priority": 0 },
        { "type": "attackMelee", "priority": 2, "speed": 1.0 },
        { "type": "moveTowardsTarget", "priority": 3, "speed": 1.0 },
        { "type": "wander", "priority": 5, "speed": 0.8 },
        { "type": "lookAtPlayer", "priority": 6, "range": 8.0 }
    ],

    "targeting": {
        "selector": "nearestPlayer",
        "range": 35.0
    },

    "drops": [
        { "item": "rotten_flesh", "count": "0-2" },
        { "item": "iron_ingot", "count": 1, "chance": 0.025 }
    ],

    "experience": { "min": 5, "max": 5 }
}
```

### Passive Mob (Cow)

```json
{
    "name": "cow",
    "displayName": "Cow",
    "category": "PASSIVE",

    "health": 10.0,
    "movementSpeed": 0.2,

    "model": "entities/cow/cow.hym",
    "hitbox": { "width": 0.9, "height": 1.4 },

    "behaviors": [
        { "type": "panic", "priority": 1, "speed": 1.25 },
        { "type": "breed", "priority": 2 },
        { "type": "tempt", "priority": 3, "items": ["wheat"] },
        { "type": "followParent", "priority": 4 },
        { "type": "wander", "priority": 5 },
        { "type": "lookAtPlayer", "priority": 6 }
    ],

    "drops": [
        { "item": "leather", "count": "0-2" },
        { "item": "raw_beef", "count": "1-3" }
    ]
}
```

### NPC

```json
{
    "name": "villager",
    "displayName": "Villager",
    "category": "NPC",

    "health": 20.0,
    "movementSpeed": 0.25,

    "model": "entities/villager/villager.hym",
    "hitbox": { "width": 0.6, "height": 1.95 },

    "behaviors": [
        { "type": "avoidEntity", "priority": 1, "target": "zombie", "range": 8 },
        { "type": "trade", "priority": 2 },
        { "type": "lookAtPlayer", "priority": 3 },
        { "type": "wander", "priority": 4 }
    ],

    "components": [
        { "type": "trader", "profession": "farmer" },
        { "type": "reputation" }
    ]
}
```

### Projectile (Arrow)

```json
{
    "name": "arrow",
    "displayName": "Arrow",
    "category": "PROJECTILE",

    "model": "entities/arrow/arrow.hym",
    "hitbox": { "width": 0.5, "height": 0.5 },

    "components": [
        {
            "type": "projectile",
            "damage": 2.0,
            "gravity": true,
            "piercing": 0
        }
    ],

    "persistent": false
}
```

## See Also

- [Asset Types](Asset-Types) - All asset types reference
- [Component System](Component-System) - ECS documentation
- [Entity System](Entity-System) - Entity management
- [Event Types](Event-Types) - Entity events
