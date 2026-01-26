
The Hytale Server uses an Entity Component System (ECS) architecture for game entities. This document covers the component system design and usage.

## Overview

The ECS is implemented in `com.hypixel.hytale.component` and follows the standard ECS pattern:

- **Entity**: A unique identifier (ID)
- **Component**: Pure data attached to entities
- **System**: Logic that processes entities with specific components

## Architecture

```
ComponentRegistry
├── Component Types
│   ├── Component (data class)
│   └── ComponentType (metadata)
├── Resources
│   ├── Resource (shared data)
│   └── ResourceType (metadata)
├── Systems
│   ├── TickingSystem (per-tick processing)
│   ├── EventSystem (event-driven processing)
│   └── DataSystem (data management)
├── Archetypes
│   ├── Archetype (entity template)
│   └── ArchetypeChunk (entity storage)
└── Queries
    └── Query (entity selection)
```

## Key Classes

### Core

| Class | Description |
|-------|-------------|
| `ComponentRegistry` | Central registry for components/systems |
| `Component` | Base interface for components |
| `ComponentType` | Metadata for component types |
| `Archetype` | Entity template (component composition) |
| `ArchetypeChunk` | Contiguous entity storage |
| `Ref` | Entity reference |
| `Holder` | Strong entity reference |

### Systems

| Class | Description |
|-------|-------------|
| `ISystem` | Base system interface |
| `System` | Abstract system base |
| `TickingSystem` | Per-tick processing |
| `EntityTickingSystem` | Per-entity per-tick |
| `ArchetypeTickingSystem` | Per-archetype processing |
| `EventSystem` | Event-driven systems |
| `EntityEventSystem` | Per-entity event handling |
| `WorldEventSystem` | World-level events |

### Queries

| Class | Description |
|-------|-------------|
| `Query` | Base query interface |
| `AndQuery` | Requires all components |
| `OrQuery` | Requires any component |
| `NotQuery` | Excludes components |
| `AnyQuery` | Matches any entity |
| `ExactArchetypeQuery` | Exact archetype match |

### Spatial

| Class | Description |
|-------|-------------|
| `SpatialStructure` | Spatial indexing |
| `SpatialSystem` | Spatial queries |
| `KDTree` | K-D tree implementation |
| `MortonCode` | Z-order curve encoding |

## Component Lifecycle

### Creation

```java
// Define a component
public class HealthComponent implements Component {
    private float health;
    private float maxHealth;

    public float getHealth() { return health; }
    public void setHealth(float health) { this.health = health; }
    public float getMaxHealth() { return maxHealth; }
    public void setMaxHealth(float maxHealth) { this.maxHealth = maxHealth; }
}

// Register component type
ComponentType<HealthComponent> HEALTH_TYPE =
    registry.registerComponent(HealthComponent.class, HealthComponent::new);
```

### Adding Components

```java
// Add component to entity
entity.addComponent(HEALTH_TYPE, healthComponent);

// Or via CommandBuffer for deferred execution
commandBuffer.addComponent(entityRef, HEALTH_TYPE, healthComponent);
```

### Removing Components

```java
// Remove component
entity.removeComponent(HEALTH_TYPE);

// Via CommandBuffer
commandBuffer.removeComponent(entityRef, HEALTH_TYPE);
```

## Archetypes

Archetypes define the component composition of entities:

```java
// Create archetype with specific components
Archetype playerArchetype = registry.getArchetype(
    POSITION_TYPE,
    ROTATION_TYPE,
    HEALTH_TYPE,
    INVENTORY_TYPE,
    PLAYER_DATA_TYPE
);

// Create entity with archetype
Ref entity = registry.createEntity(playerArchetype);
```

### Archetype Chunks

Entities with the same archetype are stored in contiguous memory chunks for cache efficiency:

```
ArchetypeChunk
├── Entity IDs: [0, 1, 2, 3, ...]
├── Position[]: [pos0, pos1, pos2, pos3, ...]
├── Health[]:   [hp0,  hp1,  hp2,  hp3,  ...]
└── ...
```

## Systems

### Ticking System

Processes entities each tick:

```java
public class DamageOverTimeSystem extends EntityTickingSystem {
    private final ComponentType<HealthComponent> healthType;
    private final ComponentType<DotComponent> dotType;

    @Override
    public Query getQuery() {
        return Query.and(healthType, dotType);
    }

    @Override
    public void tick(Ref entity, float deltaTime) {
        HealthComponent health = entity.get(healthType);
        DotComponent dot = entity.get(dotType);

        health.setHealth(health.getHealth() - dot.getDamage() * deltaTime);
    }
}
```

### Event System

Responds to ECS events:

```java
public class DeathSystem extends EntityEventSystem<DeathEvent> {
    @Override
    public void onEvent(Ref entity, DeathEvent event) {
        // Handle entity death
        dropLoot(entity);
        playDeathAnimation(entity);
        scheduleRemoval(entity);
    }
}
```

### Archetype System

Processes all entities of an archetype at once:

```java
public class PhysicsSystem extends ArchetypeTickingSystem {
    @Override
    public void tick(ArchetypeChunk chunk, float deltaTime) {
        Position[] positions = chunk.getArray(positionType);
        Velocity[] velocities = chunk.getArray(velocityType);

        for (int i = 0; i < chunk.getCount(); i++) {
            positions[i].add(velocities[i].mul(deltaTime));
        }
    }
}
```

## Queries

### Basic Queries

```java
// All entities with Health AND Position
Query query = Query.and(HEALTH_TYPE, POSITION_TYPE);

// All entities with Health OR Shield
Query query = Query.or(HEALTH_TYPE, SHIELD_TYPE);

// All entities with Health but NOT Invincible
Query query = Query.and(HEALTH_TYPE, Query.not(INVINCIBLE_TYPE));
```

### Query Execution

```java
// Iterate matching entities
registry.query(query).forEach(entity -> {
    // Process entity
});

// With components
registry.query(query).forEach((entity, health, position) -> {
    // Process with direct component access
});
```

## Dependencies

Systems can declare dependencies for ordering:

```java
public class MySystem extends TickingSystem {
    @Override
    public List<Dependency> getDependencies() {
        return List.of(
            Dependency.after(PhysicsSystem.class),
            Dependency.before(RenderSystem.class)
        );
    }
}
```

### Dependency Types

| Type | Description |
|------|-------------|
| `SystemDependency` | Depends on specific system |
| `SystemTypeDependency` | Depends on system type |
| `SystemGroupDependency` | Depends on system group |
| `RootDependency` | Special root dependency |

### Ordering

```java
public enum Order {
    BEFORE,  // Run before dependency
    AFTER    // Run after dependency
}

public enum OrderPriority {
    SOFT,    // Preference
    HARD     // Requirement
}
```

## Resources

Shared data accessible by systems:

```java
// Define resource
public class GameTimeResource implements Resource {
    private float totalTime;
    private float deltaTime;
    // getters/setters
}

// Register and access
ResourceType<GameTimeResource> TIME_TYPE =
    registry.registerResource(GameTimeResource.class, GameTimeResource::new);

GameTimeResource time = registry.getResource(TIME_TYPE);
```

## Command Buffer

Deferred operations for thread safety:

```java
CommandBuffer buffer = registry.getCommandBuffer();

// Queue operations
buffer.createEntity(archetype);
buffer.addComponent(ref, componentType, component);
buffer.removeComponent(ref, componentType);
buffer.destroyEntity(ref);

// Execute at safe point
buffer.execute();
```

## Spatial Queries

For location-based queries:

```java
SpatialStructure spatial = world.getSpatialStructure();

// Find entities near position
List<Ref> nearby = spatial.query(position, radius);

// Find closest entity
Ref closest = spatial.findClosest(position, maxDistance);

// K-D Tree for efficient spatial queries
KDTree<Ref> tree = spatial.getTree();
tree.insert(position, entity);
tree.findNearest(queryPosition, k);
```

## Serialization

Components can be serialized:

```java
// NonSerialized annotation to exclude
@NonSerialized
public class TransientComponent implements Component {
    // Not saved
}

// Use Codec for serialization
public class SerializedComponent implements Component {
    public static final Codec<SerializedComponent> CODEC = // ...
}
```

## Best Practices

1. **Keep components small** - Just data, no logic
2. **Use appropriate systems** - Ticking vs Event-driven
3. **Batch operations** - Use CommandBuffer for bulk changes
4. **Query efficiently** - Cache queries, use appropriate query types
5. **Leverage archetypes** - Group related components
6. **Use spatial structures** - For location-based queries
7. **Declare dependencies** - Ensure correct system ordering
8. **Profile performance** - Use system metrics

## Entity Events

### ECS Event Types

| Event | Description |
|-------|-------------|
| `AddReason` | Why entity was added |
| `RemoveReason` | Why entity was removed |
| `EntityEventType` | Per-entity event type |
| `WorldEventType` | World-level event type |

```java
public enum AddReason {
    SPAWN,
    LOAD,
    TRANSFER,
    // ...
}

public enum RemoveReason {
    DESPAWN,
    DEATH,
    UNLOAD,
    TRANSFER,
    // ...
}
```

## Metrics

System performance is tracked:

```java
SystemMetricData metrics = system.getMetrics();
long processingTime = metrics.getProcessingTime();
int entitiesProcessed = metrics.getEntitiesProcessed();
```
