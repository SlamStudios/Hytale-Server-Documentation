# Component System (ECS)

The Hytale Server uses an Entity Component System (ECS) architecture for game entities. This document covers the component system design and usage.

## Overview

The ECS is implemented in `com.hypixel.hytale.component` and follows the standard ECS pattern:

- **Entity**: A unique reference (`Ref`) pointing to component data
- **Component**: Pure data attached to entities
- **System**: Logic that processes entities with specific components
- **Archetype**: Template defining entity component composition

## Architecture

```
ComponentRegistry
├── Component Types
│   ├── Component<ECS_TYPE> (data interface)
│   └── ComponentType<ECS_TYPE, T> (metadata + query)
├── Store<ECS_TYPE>
│   ├── ArchetypeChunk (contiguous entity storage)
│   ├── CommandBuffer (deferred operations)
│   └── ParallelTask (parallel processing)
├── Systems
│   ├── TickingSystem (per-tick processing)
│   ├── EntityTickingSystem (per-entity per-tick)
│   ├── ArchetypeTickingSystem (per-archetype processing)
│   ├── EntityEventSystem (entity events)
│   └── WorldEventSystem (world events)
├── Archetypes
│   └── Archetype<ECS_TYPE> (implements Query)
├── Queries
│   ├── Query<ECS_TYPE> (interface)
│   ├── AndQuery / OrQuery / NotQuery
│   └── ExactArchetypeQuery
└── Resources
    └── Resource<ECS_TYPE> (shared data)
```

## Key Classes

| Class | Description |
|-------|-------------|
| `ComponentRegistry<ECS_TYPE>` | Central registry for components/systems |
| `Component<ECS_TYPE>` | Base interface for components |
| `ComponentType<ECS_TYPE, T>` | Metadata for component types, also a Query |
| `Archetype<ECS_TYPE>` | Entity template (component composition), also a Query |
| `ArchetypeChunk<ECS_TYPE>` | Contiguous entity storage |
| `Ref<ECS_TYPE>` | Entity reference |
| `Holder<ECS_TYPE>` | Entity holder with components (for creation/transfer) |
| `Store<ECS_TYPE>` | Entity storage and system execution |
| `CommandBuffer<ECS_TYPE>` | Deferred operations buffer |

---

## Defining Components

Components are pure data classes implementing the `Component<ECS_TYPE>` interface:

```java
public class HealthComponent implements Component<EntityStore> {
    private float health;
    private float maxHealth;
    
    public HealthComponent() {
        this.health = 20.0f;
        this.maxHealth = 20.0f;
    }
    
    public float getHealth() { return health; }
    public void setHealth(float health) { this.health = health; }
    public float getMaxHealth() { return maxHealth; }
    public void setMaxHealth(float maxHealth) { this.maxHealth = maxHealth; }
    
    @Override
    @Nonnull
    public Component<EntityStore> clone() {
        HealthComponent copy = new HealthComponent();
        copy.health = this.health;
        copy.maxHealth = this.maxHealth;
        return copy;
    }
}
```

### Registering Components in a System

Components are typically registered within a System class:

```java
public class HealthSystem extends EntityTickingSystem<EntityStore> {
    
    // Register component type - this is the handle used to access the component
    private final ComponentType<EntityStore, HealthComponent> healthType = 
        registerComponent(HealthComponent.class, HealthComponent::new);
    
    // With serialization codec for persistence
    private final ComponentType<EntityStore, HealthComponent> healthType = 
        registerComponent(HealthComponent.class, "Health", HealthComponent.CODEC);
    
    @Override
    public Query<EntityStore> getQuery() {
        // healthType itself IS a Query (checks if entity has this component)
        return healthType;
    }
    
    @Override
    public void tick(float dt, int index, 
                     @Nonnull ArchetypeChunk<EntityStore> chunk,
                     @Nonnull Store<EntityStore> store,
                     @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        // Get component from chunk at index
        HealthComponent health = chunk.getComponent(healthType, index);
        
        // Process health regeneration
        if (health.getHealth() < health.getMaxHealth()) {
            health.setHealth(Math.min(health.getHealth() + dt, health.getMaxHealth()));
        }
    }
}
```

---

## Entity References (Ref)

Entities are referenced using `Ref<ECS_TYPE>`:

```java
Ref<EntityStore> entityRef;

// Check if reference is still valid
if (entityRef.isValid()) {
    // Use reference
    Store<EntityStore> store = entityRef.getStore();
    int index = entityRef.getIndex();
}

// Validate (throws if invalid)
entityRef.validate();
```

---

## Entity Holders

`Holder<ECS_TYPE>` is used when creating entities or transferring them:

```java
// Create a new holder from the registry
Holder<EntityStore> holder = registry.newHolder();

// Add components to holder
holder.addComponent(healthType, new HealthComponent());
holder.addComponent(positionType, new PositionComponent());

// Or ensure component exists (creates with default if missing)
holder.ensureComponent(healthType);

// Get component from holder
HealthComponent health = holder.getComponent(healthType);

// Put component (add or replace)
holder.putComponent(healthType, newHealth);

// Remove component
holder.removeComponent(healthType);

// Clone the holder
Holder<EntityStore> copy = holder.clone();
```

---

## Archetypes

Archetypes define the component composition of entities and also serve as queries:

```java
// Create archetype with specific components
Archetype<EntityStore> playerArchetype = Archetype.of(
    positionType,
    rotationType,
    healthType,
    inventoryType,
    playerDataType
);

// Check if archetype contains a component type
boolean hasHealth = playerArchetype.contains(healthType);

// Archetypes ARE queries - use directly
Query<EntityStore> query = playerArchetype;  // Matches entities with ALL these components

// Add component to archetype (creates new archetype)
Archetype<EntityStore> newArchetype = Archetype.add(playerArchetype, effectsType);

// Remove component from archetype
Archetype<EntityStore> reducedArchetype = Archetype.remove(playerArchetype, inventoryType);
```

---

## Queries

Queries select entities based on component criteria. `ComponentType` and `Archetype` both implement `Query`.

### Using ComponentType as Query

```java
// A single ComponentType IS a query - matches entities that have this component
Query<EntityStore> query = healthType;
```

### Query Combinators

```java
// All entities with Health AND Position
Query<EntityStore> query = Query.and(healthType, positionType);

// All entities with Health OR Shield
Query<EntityStore> query = Query.or(healthType, shieldType);

// All entities with Health but NOT Invincible
Query<EntityStore> query = Query.and(healthType, Query.not(invincibleType));

// Match any entity
Query<EntityStore> anyQuery = Query.any();

// Exact archetype match (must have exactly these components, no more)
Query<EntityStore> exactQuery = playerArchetype.asExactQuery();
```

### Query in Systems

```java
public class DamageOverTimeSystem extends EntityTickingSystem<EntityStore> {
    private final ComponentType<EntityStore, HealthComponent> healthType;
    private final ComponentType<EntityStore, DotComponent> dotType;
    
    @Override
    public Query<EntityStore> getQuery() {
        // Process entities that have BOTH Health AND DoT components
        return Query.and(healthType, dotType);
    }
    
    @Override
    public void tick(float dt, int index, 
                     @Nonnull ArchetypeChunk<EntityStore> chunk,
                     @Nonnull Store<EntityStore> store,
                     @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        HealthComponent health = chunk.getComponent(healthType, index);
        DotComponent dot = chunk.getComponent(dotType, index);
        
        health.setHealth(health.getHealth() - dot.getDamage() * dt);
    }
}
```

---

## Systems

### TickingSystem

Base class for systems that run every tick:

```java
public abstract class TickingSystem<ECS_TYPE> extends System<ECS_TYPE> 
        implements TickableSystem<ECS_TYPE> {
    
    // Called every tick with delta time, system index, and store
    public abstract void tick(float dt, int systemIndex, @Nonnull Store<ECS_TYPE> store);
}
```

### ArchetypeTickingSystem

Processes matching archetype chunks:

```java
public class PhysicsSystem extends ArchetypeTickingSystem<EntityStore> {
    private final ComponentType<EntityStore, PositionComponent> positionType;
    private final ComponentType<EntityStore, VelocityComponent> velocityType;
    
    @Override
    public Query<EntityStore> getQuery() {
        return Query.and(positionType, velocityType);
    }
    
    @Override
    public void tick(float dt, 
                     @Nonnull ArchetypeChunk<EntityStore> chunk,
                     @Nonnull Store<EntityStore> store,
                     @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        // Process all entities in this chunk
        int size = chunk.size();
        for (int i = 0; i < size; i++) {
            PositionComponent pos = chunk.getComponent(positionType, i);
            VelocityComponent vel = chunk.getComponent(velocityType, i);
            
            pos.add(vel.getX() * dt, vel.getY() * dt, vel.getZ() * dt);
        }
    }
}
```

### EntityTickingSystem

Processes entities one at a time (simpler API):

```java
public class HealthRegenSystem extends EntityTickingSystem<EntityStore> {
    private final ComponentType<EntityStore, HealthComponent> healthType;
    
    @Override
    public Query<EntityStore> getQuery() {
        return healthType;
    }
    
    @Override
    public void tick(float dt, int index,
                     @Nonnull ArchetypeChunk<EntityStore> chunk,
                     @Nonnull Store<EntityStore> store,
                     @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        HealthComponent health = chunk.getComponent(healthType, index);
        
        if (health.getHealth() < health.getMaxHealth()) {
            float regen = 0.5f * dt;  // 0.5 health per second
            health.setHealth(Math.min(health.getHealth() + regen, health.getMaxHealth()));
        }
    }
    
    // Optional: enable parallel processing for large entity counts
    @Override
    public boolean isParallel(int archetypeChunkSize, int taskCount) {
        return archetypeChunkSize > 100;  // Parallelize if >100 entities
    }
}
```

### EntityEventSystem

Responds to entity-specific events:

```java
public class DeathEventSystem extends EntityEventSystem<EntityStore, DeathEvent> {
    
    @Override
    public void onEvent(@Nonnull Ref<EntityStore> entityRef, 
                        @Nonnull DeathEvent event,
                        @Nonnull Store<EntityStore> store,
                        @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        // Handle entity death
        dropLoot(entityRef, store);
        playDeathEffect(entityRef);
        
        // Schedule entity removal
        commandBuffer.removeEntity(entityRef, RemoveReason.DEATH);
    }
}
```

### WorldEventSystem

Responds to world-level events:

```java
public class WorldLoadEventSystem extends WorldEventSystem<EntityStore, WorldLoadEvent> {
    
    @Override
    public void onEvent(@Nonnull WorldLoadEvent event,
                        @Nonnull Store<EntityStore> store,
                        @Nonnull CommandBuffer<EntityStore> commandBuffer) {
        // Handle world load
        initializeWorldEntities(event.getWorld(), store, commandBuffer);
    }
}
```

---

## CommandBuffer

Deferred operations for thread safety and batching:

```java
// Get command buffer from store
CommandBuffer<EntityStore> buffer = store.takeCommandBuffer();

try {
    // Queue entity creation
    Holder<EntityStore> holder = registry.newHolder();
    holder.addComponent(positionType, new PositionComponent(x, y, z));
    holder.addComponent(healthType, new HealthComponent());
    buffer.addEntity(holder, AddReason.SPAWN);
    
    // Queue component operations
    buffer.addComponent(existingRef, effectType, new PoisonEffect());
    buffer.removeComponent(existingRef, shieldType);
    
    // Queue entity removal
    buffer.removeEntity(deadEntityRef, RemoveReason.DEATH);
    
    // Execute all queued operations
    buffer.execute();
    
} finally {
    // Return buffer to pool
    store.storeCommandBuffer(buffer);
}
```

### In System Tick Methods

Systems receive a CommandBuffer as a parameter:

```java
@Override
public void tick(float dt, int index,
                 @Nonnull ArchetypeChunk<EntityStore> chunk,
                 @Nonnull Store<EntityStore> store,
                 @Nonnull CommandBuffer<EntityStore> commandBuffer) {
    
    HealthComponent health = chunk.getComponent(healthType, index);
    
    if (health.getHealth() <= 0) {
        Ref<EntityStore> ref = chunk.getRef(index);
        // Queue removal - will be processed after all systems tick
        commandBuffer.removeEntity(ref, RemoveReason.DEATH);
    }
}
```

---

## Resources

Shared data accessible by all systems:

```java
public class GameTimeResource implements Resource<EntityStore> {
    private float totalTime;
    private float deltaTime;
    
    public float getTotalTime() { return totalTime; }
    public void setTotalTime(float time) { this.totalTime = time; }
    public float getDeltaTime() { return deltaTime; }
    public void setDeltaTime(float dt) { this.deltaTime = dt; }
    
    @Override
    public Resource<EntityStore> clone() {
        GameTimeResource copy = new GameTimeResource();
        copy.totalTime = this.totalTime;
        copy.deltaTime = this.deltaTime;
        return copy;
    }
}

// Register in system
public class TimeSystem extends TickingSystem<EntityStore> {
    private final ResourceType<EntityStore, GameTimeResource> timeResource = 
        registerResource(GameTimeResource.class, GameTimeResource::new);
    
    @Override
    public void tick(float dt, int systemIndex, @Nonnull Store<EntityStore> store) {
        GameTimeResource time = store.getResourceStorage().getResource(timeResource);
        time.setDeltaTime(dt);
        time.setTotalTime(time.getTotalTime() + dt);
    }
}
```

---

## Store Operations

The `Store<ECS_TYPE>` manages entities and executes systems:

```java
Store<EntityStore> store = // ...

// Get entity count
int count = store.getEntityCount();

// Access archetype chunks
ArchetypeChunk<EntityStore>[] chunks = store.getArchetypeChunks();

// Get component from entity reference
HealthComponent health = store.getComponent(entityRef, healthType);

// Get archetype for an entity
Archetype<EntityStore> archetype = store.getArchetype(entityRef);

// Copy entity to holder
Holder<EntityStore> holder = store.copyEntity(entityRef);
```

---

## ArchetypeChunk Operations

Chunks store entities with the same archetype contiguously:

```java
ArchetypeChunk<EntityStore> chunk = // ...

// Get chunk size
int size = chunk.size();

// Get entity reference at index
Ref<EntityStore> ref = chunk.getRef(index);

// Get component at index
HealthComponent health = chunk.getComponent(healthType, index);

// Check if chunk's archetype contains a component
boolean hasHealth = chunk.getArchetype().contains(healthType);
```

---

## Add/Remove Reasons

Track why entities are added or removed:

```java
public enum AddReason {
    SPAWN,      // Newly spawned
    LOAD,       // Loaded from storage
    TRANSFER,   // Transferred from another store
    // ...
}

public enum RemoveReason {
    DESPAWN,    // Normal despawn
    DEATH,      // Entity died
    UNLOAD,     // Unloaded to storage
    TRANSFER,   // Transferred to another store
    REMOVE,     // Generic removal
    // ...
}
```

---

## Component Serialization

Components can be serialized for persistence:

```java
public class HealthComponent implements Component<EntityStore> {
    // Codec for serialization
    public static final BuilderCodec<HealthComponent> CODEC = BuilderCodec
        .builder(HealthComponent.class, HealthComponent::new)
        .append("health", Codec.FLOAT, HealthComponent::getHealth, HealthComponent::setHealth)
        .append("maxHealth", Codec.FLOAT, HealthComponent::getMaxHealth, HealthComponent::setMaxHealth)
        .build();
    
    // ... rest of component
}

// Register with codec
ComponentType<EntityStore, HealthComponent> healthType = 
    registerComponent(HealthComponent.class, "Health", HealthComponent.CODEC);
```

### NonSerialized Annotation

Mark components that shouldn't be saved:

```java
@NonSerialized
public class TransientComponent implements Component<EntityStore> {
    // This component won't be persisted
}
```

---

## Best Practices

1. **Keep components small** - Pure data, no behavior logic
2. **Use appropriate systems** - `EntityTickingSystem` for simple per-entity logic, `ArchetypeTickingSystem` for bulk processing
3. **Batch operations** - Use `CommandBuffer` for entity creation/removal during ticks
4. **Query efficiently** - Cache queries, use `ComponentType` directly when checking for single component
5. **Leverage archetypes** - Group commonly-used components together
6. **Enable parallelism** - Override `isParallel()` for CPU-intensive systems with many entities
7. **Use resources** - For shared data that multiple systems need
8. **Handle entity removal** - Always use `CommandBuffer.removeEntity()` during system ticks

---

## Example: Complete Damage System

```java
public class DamageSystem extends System<EntityStore> {
    
    // Component types
    private final ComponentType<EntityStore, HealthComponent> healthType = 
        registerComponent(HealthComponent.class, "Health", HealthComponent.CODEC);
    
    private final ComponentType<EntityStore, DamageQueueComponent> damageQueueType = 
        registerComponent(DamageQueueComponent.class, DamageQueueComponent::new);
    
    // Inner system to process damage
    public class ProcessDamageSystem extends EntityTickingSystem<EntityStore> {
        
        @Override
        public Query<EntityStore> getQuery() {
            return Query.and(healthType, damageQueueType);
        }
        
        @Override
        public void tick(float dt, int index,
                         @Nonnull ArchetypeChunk<EntityStore> chunk,
                         @Nonnull Store<EntityStore> store,
                         @Nonnull CommandBuffer<EntityStore> commandBuffer) {
            
            HealthComponent health = chunk.getComponent(healthType, index);
            DamageQueueComponent queue = chunk.getComponent(damageQueueType, index);
            
            // Process all queued damage
            for (DamageInstance damage : queue.drain()) {
                health.setHealth(health.getHealth() - damage.getAmount());
            }
            
            // Check for death
            if (health.getHealth() <= 0) {
                Ref<EntityStore> ref = chunk.getRef(index);
                commandBuffer.removeEntity(ref, RemoveReason.DEATH);
            }
        }
    }
}
```
