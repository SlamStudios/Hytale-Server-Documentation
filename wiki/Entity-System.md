
The Hytale Server's entity system manages all game entities including players, NPCs, items, and projectiles. This document covers entity architecture, components, and behaviors.

## Overview

The entity system is implemented in:
- `com.hypixel.hytale.server.core.entity` - Core entity classes
- `com.hypixel.hytale.component` - ECS foundation

## Architecture

```
Entity System
├── Entity (base class)
│   ├── LivingEntity
│   │   └── Player
│   ├── BlockEntity
│   └── Custom entities
├── Components
│   ├── Health/Damage
│   ├── Movement
│   ├── Effects
│   └── Custom components
├── Systems
│   ├── Ticking systems
│   ├── Event systems
│   └── Data systems
└── Managers
    ├── InteractionManager
    ├── MovementManager
    └── EffectController
```

## Key Classes

### Core Entities

| Class | Location | Description |
|-------|----------|-------------|
| `Entity` | `server.core.entity` | Base entity class |
| `LivingEntity` | `server.core.entity` | Living creature base |
| `Player` | `server.core.entity.entities` | Player entity |
| `BlockEntity` | `server.core.entity.entities` | Block-attached entity |

### Components

| Class | Description |
|-------|-------------|
| `HealthComponent` | Health/damage data |
| `DamageDataComponent` | Damage information |
| `KnockbackComponent` | Knockback state |
| `MovementStatesComponent` | Movement state |
| `EffectControllerComponent` | Active effects |
| `ProjectileComponent` | Projectile behavior |
| `UUIDComponent` | Unique identifier |

### Managers

| Class | Description |
|-------|-------------|
| `InteractionManager` | Entity interactions |
| `MovementManager` | Player movement |
| `StatModifiersManager` | Stat modifiers |
| `WindowManager` | UI windows |
| `PageManager` | UI pages |
| `HudManager` | HUD elements |

## Entity Class

Base class for all entities. Entities are ECS components stored in `EntityStore`.

```java
public abstract class Entity implements Component<EntityStore> {
    // Entity reference in the ECS
    @Nullable
    protected Ref<EntityStore> reference;
    
    // Network ID for client sync
    protected int networkId;
    
    // World the entity is in
    @Nullable
    protected World world;
    
    // Default animations enum
    public static class DefaultAnimations {
        // IDLE, WALK, RUN, JUMP, FALL, ATTACK, etc.
    }
}
```

**Note:** Entity access is typically done through the ECS component system rather than direct method calls. Use `ComponentAccessor` and `ComponentType` for component access.

## Living Entity

Base for creatures with health (extends Entity):

```java
public abstract class LivingEntity extends Entity {
    // LivingEntity has its own BuilderCodec for serialization
    public static final BuilderCodec<LivingEntity> CODEC;
}
```

**Note:** Health, damage, and effects are managed through ECS components (e.g., `HealthComponent`, `DamageDataComponent`, `EffectControllerComponent`) rather than direct methods on LivingEntity.

## Player Entity

Player implementation (extends LivingEntity, implements CommandSender, PermissionHolder):

```java
public class Player extends LivingEntity implements CommandSender, PermissionHolder {
    // Get the player's UUID
    @Nonnull
    public UUID getUuid();
    
    // Get the player's username  
    @Nonnull
    public String getUsername();
    
    // Network connection
    @Nonnull
    public PacketHandler getPacketHandler();
    
    public void sendPacket(@Nonnull Packet packet);
    
    // Managers (all @Nonnull)
    public WindowManager getWindowManager();
    public PageManager getPageManager();
    public HudManager getHudManager();
    public HotbarManager getHotbarManager();
    public CameraManager getCameraManager();
    public MovementManager getMovementManager();
    
    // Game mode
    public GameMode getGameMode();
    public void setGameMode(@Nonnull GameMode gameMode);
    
    // CommandSender implementation
    @Override
    public void sendMessage(@Nonnull Message message);
    
    @Override  
    public boolean hasPermission(@Nonnull String permission);
    
    // Inventory access
    @Nullable
    public Inventory getInventory();
    
    @Nullable
    public ItemStack getHeldItem();
}
```

## Player Components

### Player Data

```java
// Config data
PlayerConfigData configData;

// Death position
PlayerDeathPositionData deathPosition;

// Respawn point
PlayerRespawnPointData respawnPoint;

// World-specific data
PlayerWorldData worldData;

// Item usage tracking
UniqueItemUsagesComponent itemUsages;
```

### Movement

```java
MovementConfig config;
MovementManager manager;
MovementStatesComponent states;

// Movement states
states.isRunning();
states.isCrouching();
states.isJumping();
states.isFlying();
```

## Entity Effects

Active effects on entities:

```java
public class ActiveEntityEffect {
    EntityEffect effect;
    float duration;
    float strength;
}

// Effect controller
EffectControllerComponent controller;
controller.addEffect(effect);
controller.removeEffect(effect);
controller.getActiveEffects();
```

## Damage System

### Damage Data

```java
DamageDataComponent damageData;

// Damage info
float amount;
DamageSource source;
Entity attacker;
```

### Knockback

```java
KnockbackComponent knockback;

// Knockback systems
KnockbackSystems.ApplyKnockback
KnockbackSystems.ApplyPlayerKnockback
```

## Entity Groups

Organizing entities:

```java
EntityGroup group;

// Group operations
group.addEntity(entity);
group.removeEntity(entity);
group.forEachEntity(consumer);
```

## References

### Ref Types

| Type | Description |
|------|-------------|
| `Ref` | Basic entity reference |
| `PersistentRef` | Survives entity unload |
| `InvalidatablePersistentRef` | Can be invalidated |

```java
// Persistent reference
PersistentRef ref = entity.getPersistentRef();
Entity entity = ref.get();

// Reference counting
PersistentRefCount refCount;
```

## Interactions

### InteractionManager

Manages entity interactions:

```java
InteractionManager manager;

// Interaction chain
InteractionChain chain = manager.createChain();
chain.add(interaction);
chain.execute(context);
```

### InteractionContext

```java
public class InteractionContext {
    Entity source;
    Entity target;
    World world;
    // Snapshot for rollback
    SnapshotProvider snapshotProvider;
}
```

### InteractionChain

```java
InteractionChain
├── TempChain (temporary)
└── CallState (execution state)
```

## Entity Systems

### Movement Systems

```java
MovementStatesSystems
├── AddSystem      // Initialize on add
├── PlayerInitSystem // Player-specific init
└── TickingSystem  // Per-tick update
```

### Knockback Systems

```java
KnockbackSystems
├── ApplyKnockback       // Generic entities
└── ApplyPlayerKnockback // Player-specific
```

### Damage Systems

```java
DamageDataSetupSystem
└── Initialize damage tracking
```

### Nameplate Systems

```java
NameplateSystems
├── EntityTrackerUpdate // Update nameplates
└── EntityTrackerRemove // Remove nameplates
```

## Player UI

### Windows

```java
WindowManager manager = player.getWindowManager();

// Window types
Window window;
ContainerWindow containerWindow;
BlockWindow blockWindow;
ValidatedWindow validatedWindow;

// Events
Window.WindowCloseEvent closeEvent;
```

### Pages

```java
PageManager manager = player.getPageManager();

// Page types
CustomUIPage page;
BasicCustomUIPage basicPage;
InteractiveCustomUIPage interactivePage;
RespawnPage respawnPage;
ChoiceBasePage choicePage;
```

### HUD

```java
HudManager manager = player.getHudManager();

// Custom HUD
CustomUIHud hud;
```

## Camera

```java
CameraManager camera = player.getCameraManager();

// Camera control
camera.setPosition(position);
camera.setRotation(yaw, pitch);
camera.shake(intensity, duration);
```

## Hotbar

```java
HotbarManager hotbar = player.getHotbarManager();

// Hotbar access
int slot = hotbar.getSelectedSlot();
ItemStack item = hotbar.getItem(slot);
hotbar.setSelectedSlot(slot);
```

## Hidden Players

```java
HiddenPlayersManager hidden;

// Hide/show players
hidden.hidePlayer(target);
hidden.showPlayer(target);
hidden.isHidden(target);
```

## Entity Snapshots

For rollback/validation:

```java
EntitySnapshot snapshot = entity.createSnapshot();

// Restore state
entity.restoreSnapshot(snapshot);
```

## Explosions

```java
ExplosionConfig config;
ExplosionUtils.explode(world, position, config);

// Config options
float radius;
float damage;
boolean breakBlocks;
boolean causeFire;
```

## Frozen State

Prevent entity updates:

```java
Frozen frozen;

// Freeze entity
entity.setFrozen(true);
boolean isFrozen = entity.isFrozen();
```

## Entity Registry

Registering entity types:

```java
EntityRegistry registry = plugin.getEntityRegistry();

// Register entity type
registry.register("my_entity", MyEntity.class, MyEntity::new);
```

## Animation

```java
AnimationUtils
├── playAnimation(entity, animation)
├── stopAnimation(entity, animation)
└── setAnimationSpeed(entity, animation, speed)
```

## Chain Synchronization

```java
ChainSyncStorage storage;

// Sync entity state across systems
```

## Best Practices

1. **Use components** - Don't subclass for data
2. **Leverage ECS** - Use systems for behavior
3. **Handle references** - Use PersistentRef for long-term
4. **Validate interactions** - Security first
5. **Batch updates** - Reduce network traffic
6. **Clean up** - Remove references on entity removal
7. **Use snapshots** - For rollback capability
