
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

Handlers are executed in priority order (lower values execute first):

```java
public enum EventPriority {
    LOWEST,    // First to run, can set initial state
    LOW,       // Early processing
    NORMAL,    // Default priority
    HIGH,      // Late processing
    HIGHEST,   // Almost last
    MONITOR    // Observe only - runs last, should NOT modify event
}
```

**Important:** `MONITOR` priority handlers should only observe events, never cancel or modify them. Use for logging, analytics, etc.

## Registering Event Handlers

### Via Plugin's EventRegistry (Recommended)

Plugins should use their `EventRegistry` for automatic cleanup on disable:

```java
public class MyPlugin extends JavaPlugin {
    @Override
    protected void setup() {
        EventRegistry events = getEventRegistry();
        
        // Simple registration (NORMAL priority)
        events.register(PlayerConnectEvent.class, this::onPlayerConnect);
        
        // With priority
        events.register(EventPriority.HIGH, PlayerChatEvent.class, this::onPlayerChat);
        
        // With custom numeric priority
        events.register((short)50, BootEvent.class, event -> {
            // Handle event
        });
    }
    
    private void onPlayerConnect(PlayerConnectEvent event) {
        getLogger().info("Player connecting...");
    }
    
    private void onPlayerChat(PlayerChatEvent event) {
        // Handle chat
    }
}
```

### Direct EventBus Registration

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

## Plugin Event Registry Benefits

Using `getEventRegistry()` instead of direct `EventBus` access provides:

1. **Automatic cleanup** - Handlers are unregistered when plugin disables
2. **State validation** - Prevents registration when plugin is disabled
3. **Lifecycle scoping** - Handlers tied to plugin lifecycle
4. **Thread safety** - Proper synchronization for plugin state

```java
// All these registrations are automatically cleaned up on plugin disable:
getEventRegistry().register(Event1.class, this::handle1);
getEventRegistry().register(Event2.class, this::handle2);
getEventRegistry().registerGlobal(KeyedEvent.class, this::handleAll);
```

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

Events related to player actions and connections. See [Event Types](Event-Types) for the full list including:
- `PlayerConnectEvent` / `PlayerDisconnectEvent`
- `PlayerReadyEvent`
- `PlayerChatEvent`
- `PlayerInteractEvent`
- `AddPlayerToWorldEvent` / `DrainPlayerFromWorldEvent`

### World Events

Events related to world and chunk operations.

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

## Example: Using PlayerChatEvent

The actual `PlayerChatEvent` from the server (this is an async event):

```java
// PlayerChatEvent is IAsyncEvent<String>, ICancellable
// Register with registerAsync
eventRegistry.registerAsync(EventPriority.LOW, PlayerChatEvent.class, future -> {
    return future.thenApply(event -> {
        // Filter content
        String filtered = filterBadWords(event.getContent());
        event.setContent(filtered);
        return event;
    });
});

eventRegistry.registerAsync(EventPriority.NORMAL, PlayerChatEvent.class, future -> {
    return future.thenApply(event -> {
        // Check mute status
        if (isMuted(event.getSender())) {
            event.setCancelled(true);
        }
        return event;
    });
});

eventRegistry.registerAsync(EventPriority.MONITOR, PlayerChatEvent.class, future -> {
    return future.thenApply(event -> {
        // Log chat (don't modify at MONITOR priority)
        logChat(event.getSender(), event.getContent());
        return event;
    });
});
```

## Example: Creating a Custom Sync Event

```java
// 1. Define the event
public class CustomActionEvent implements IEvent<Void>, ICancellable {
    private final PlayerRef player;
    private String data;
    private boolean cancelled;

    public CustomActionEvent(@Nonnull PlayerRef player, @Nonnull String data) {
        this.player = player;
        this.data = data;
    }

    @Nonnull public PlayerRef getPlayer() { return player; }
    @Nonnull public String getData() { return data; }
    public void setData(@Nonnull String data) { this.data = data; }
    
    @Override
    public boolean isCancelled() { return cancelled; }
    
    @Override
    public void setCancelled(boolean cancelled) { this.cancelled = cancelled; }
}

// 2. Register handlers
eventRegistry.register(EventPriority.NORMAL, CustomActionEvent.class, event -> {
    if (shouldBlock(event.getPlayer())) {
        event.setCancelled(true);
    }
});

// 3. Dispatch the event
CustomActionEvent event = new CustomActionEvent(playerRef, "action_data");
HytaleServer.get().getEventBus()
    .dispatchFor(CustomActionEvent.class)
    .dispatch(event);

if (!event.isCancelled()) {
    performAction(event.getData());
}
```
