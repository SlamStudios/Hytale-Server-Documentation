
The Hytale Server uses a powerful event-driven architecture for communication between systems. This document covers the event bus, event types, and how to work with events.

## Overview

The event system is implemented in `com.hypixel.hytale.event` and provides:

- Synchronous and asynchronous event handling
- Priority-based event ordering
- Keyed event filtering
- Cancellable events
- Global and specific listeners

## Architecture

```
EventBus
├── SyncEventBusRegistry (IEvent)
│   ├── EventConsumerMap
│   └── EventDispatcher
├── AsyncEventBusRegistry (IAsyncEvent)
│   ├── AsyncEventConsumer
│   └── CompletableFuture-based dispatch
└── EventRegistry (per-plugin)
```

## Key Classes

| Class | Description |
|-------|-------------|
| `EventBus` | Central event dispatcher |
| `IEvent<K>` | Synchronous event interface |
| `IAsyncEvent<K>` | Asynchronous event interface |
| `IBaseEvent<K>` | Base event interface |
| `ICancellable` | Interface for cancellable events |
| `EventPriority` | Event handler priority |
| `EventRegistration` | Registered handler reference |
| `EventRegistry` | Plugin-scoped event registration |

## Event Types

### Synchronous Events (IEvent)

Standard events that are processed immediately and block until all handlers complete.

```java
public interface IEvent<KeyType> extends IBaseEvent<KeyType> {
    // Synchronous event marker
}
```

### Asynchronous Events (IAsyncEvent)

Events that can be processed asynchronously using `CompletableFuture`.

```java
public interface IAsyncEvent<KeyType> extends IBaseEvent<KeyType> {
    // Async event marker
}
```

### Cancellable Events (ICancellable)

Events that can be cancelled by handlers.

```java
public interface ICancellable {
    boolean isCancelled();
    void setCancelled(boolean cancelled);
}
```

## Event Priority

Handlers are executed in priority order:

```java
public enum EventPriority {
    LOWEST((short)-200),
    LOW((short)-100),
    NORMAL((short)0),
    HIGH((short)100),
    HIGHEST((short)200),
    MONITOR((short)300);  // Observe only, don't modify
}
```

**Note:** Lower values execute first. MONITOR priority should only observe, never modify.

## Registering Event Handlers

### Basic Registration

```java
EventBus eventBus = HytaleServer.get().getEventBus();

// Simple registration
eventBus.register(MyEvent.class, event -> {
    // Handle event
});

// With priority
eventBus.register(EventPriority.HIGH, MyEvent.class, event -> {
    // Handle high-priority
});

// With numeric priority
eventBus.register((short)50, MyEvent.class, event -> {
    // Custom priority
});
```

### Keyed Registration

For events with a key type:

```java
// Register for specific key
eventBus.register(BlockBreakEvent.class, blockType, event -> {
    // Only triggered for this block type
});
```

### Global Registration

Handle all events of a type regardless of key:

```java
eventBus.registerGlobal(MyEvent.class, event -> {
    // Handle all MyEvent instances
});
```

### Unhandled Registration

Handle events that had no other handlers:

```java
eventBus.registerUnhandled(MyEvent.class, event -> {
    // Fallback handler
});
```

### Async Registration

For asynchronous events:

```java
eventBus.registerAsync(MyAsyncEvent.class, future -> {
    return future.thenCompose(event -> {
        // Async processing
        return CompletableFuture.completedFuture(event);
    });
});
```

## Dispatching Events

### Synchronous Dispatch

```java
// Create and dispatch
MyEvent event = new MyEvent();
eventBus.dispatchFor(MyEvent.class).dispatch(event);

// With key
eventBus.dispatchFor(MyEvent.class, key).dispatch(event);
```

### Asynchronous Dispatch

```java
CompletableFuture<MyAsyncEvent> future =
    eventBus.dispatchForAsync(MyAsyncEvent.class, key).dispatch(event);

future.thenAccept(processedEvent -> {
    // Event processed
});
```

## Plugin Event Registry

Plugins should use their event registry for automatic cleanup:

```java
public class MyPlugin extends JavaPlugin {
    @Override
    protected void setup() {
        EventRegistry registry = getEventRegistry();

        // Registration is tied to plugin lifecycle
        registry.register(MyEvent.class, this::handleEvent);
    }
}
```

**Benefits:**
- Automatic unregistration on plugin disable
- Plugin state validation
- Scoped to plugin lifecycle

## Server Events

### Lifecycle Events

| Event | When Fired |
|-------|------------|
| `BootEvent` | Server finished booting |
| `ShutdownEvent` | Server is shutting down |
| `LoadAssetEvent` | Assets are being loaded |

### Asset Events

| Event | Description |
|-------|-------------|
| `AssetStoreEvent` | Asset store operations |
| `AssetMonitorEvent` | Asset file changes |
| `GenerateAssetsEvent` | Asset generation |
| `LoadedAssetsEvent` | Assets loaded |
| `RemovedAssetsEvent` | Assets removed |

### Player Events

Events related to player actions, connections, etc.

### World Events

Events related to world operations, chunk loading, etc.

## Creating Custom Events

### Synchronous Event

```java
public class MyEvent implements IEvent<Void> {
    private final String data;

    public MyEvent(String data) {
        this.data = data;
    }

    public String getData() {
        return data;
    }
}
```

### Keyed Event

```java
public class BlockEvent implements IEvent<BlockType> {
    private final BlockType blockType;

    public BlockEvent(BlockType blockType) {
        this.blockType = blockType;
    }

    @Override
    public BlockType getKey() {
        return blockType;
    }
}
```

### Cancellable Event

```java
public class CancellableEvent implements IEvent<Void>, ICancellable {
    private boolean cancelled = false;

    @Override
    public boolean isCancelled() {
        return cancelled;
    }

    @Override
    public void setCancelled(boolean cancelled) {
        this.cancelled = cancelled;
    }
}
```

### Async Event

```java
public class MyAsyncEvent implements IAsyncEvent<Void> {
    // Async event data
}
```

## Event Bus Internals

### EventBusRegistry

Each event type gets its own registry:

```java
Map<Class<? extends IBaseEvent<?>>, EventBusRegistry<?, ?, ?>> registryMap
```

### Event Consumer Map

Stores handlers organized by:
- Key-specific handlers
- Global handlers
- Unhandled handlers
- Priority ordering

### Event Timing

When `timeEvents` is enabled, event processing time is tracked for debugging.

## Best Practices

1. **Use appropriate priority** - NORMAL for most cases
2. **Don't modify in MONITOR** - Only observe at MONITOR priority
3. **Check cancellation** - Respect `isCancelled()` when applicable
4. **Use plugin registry** - For automatic cleanup
5. **Keep handlers fast** - Avoid blocking operations in sync handlers
6. **Use async events** - For long-running operations
7. **Handle exceptions** - Don't let handlers crash the event bus

## Example: Complete Event Flow

```java
// 1. Define the event
public class PlayerChatEvent implements IEvent<Player>, ICancellable {
    private final Player player;
    private String message;
    private boolean cancelled;

    public PlayerChatEvent(Player player, String message) {
        this.player = player;
        this.message = message;
    }

    public Player getKey() { return player; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public boolean isCancelled() { return cancelled; }
    public void setCancelled(boolean cancelled) { this.cancelled = cancelled; }
}

// 2. Register handlers
eventBus.register(EventPriority.LOW, PlayerChatEvent.class, event -> {
    // Filter bad words
    event.setMessage(filterBadWords(event.getMessage()));
});

eventBus.register(EventPriority.NORMAL, PlayerChatEvent.class, event -> {
    // Check mute status
    if (isMuted(event.getKey())) {
        event.setCancelled(true);
    }
});

eventBus.register(EventPriority.MONITOR, PlayerChatEvent.class, event -> {
    // Log chat (don't modify here)
    logChat(event.getKey(), event.getMessage());
});

// 3. Dispatch the event
PlayerChatEvent event = new PlayerChatEvent(player, chatMessage);
eventBus.dispatchFor(PlayerChatEvent.class, player).dispatch(event);

if (!event.isCancelled()) {
    broadcastMessage(event.getMessage());
}
```
