# Event Types

This document provides a reference of event types in the Hytale Server. All events listed are verified from the decompiled source code.

## Overview

Events are dispatched through the [Event System](Event-System) to notify listeners of game occurrences. Events implement either `IEvent<K>` (synchronous) or `IAsyncEvent<K>` (asynchronous), and some implement `ICancellable`.

---

## Server Events

Located in `com.hypixel.hytale.server.core.event.events`:

### BootEvent

Fired when the server has fully booted.

**Implements:** `IEvent<Void>`

```java
public class BootEvent implements IEvent<Void> {
    // No properties
}
```

```java
eventRegistry.register(BootEvent.class, event -> {
    getLogger().info("Server has booted!");
});
```

### ShutdownEvent

Fired when the server begins shutdown.

**Implements:** `IEvent<Void>`

```java
public class ShutdownEvent implements IEvent<Void> {
    public static final short DISCONNECT_PLAYERS = -48;
    public static final short UNBIND_LISTENERS = -40;
    public static final short SHUTDOWN_WORLDS = -32;
}
```

```java
eventRegistry.register(ShutdownEvent.class, event -> {
    saveAllData();
});
```

### PrepareUniverseEvent

Fired during universe preparation.

**Location:** `com.hypixel.hytale.server.core.event.events.PrepareUniverseEvent`

---

## Player Events

Located in `com.hypixel.hytale.server.core.event.events.player`:

### PlayerConnectEvent

Fired when a player connection is established.

**Implements:** `IEvent<Void>`

| Property | Type | Description |
|----------|------|-------------|
| `holder` | `Holder<EntityStore>` | Entity holder for the player |
| `playerRef` | `PlayerRef` | Player reference |
| `world` | `World` (nullable) | Target world (can be set) |

```java
public class PlayerConnectEvent implements IEvent<Void> {
    public Holder<EntityStore> getHolder();
    public PlayerRef getPlayerRef();
    @Nullable public World getWorld();
    public void setWorld(@Nullable World world);
    
    @Deprecated
    @Nullable public Player getPlayer();  // Use getPlayerRef() instead
}
```

```java
eventRegistry.register(PlayerConnectEvent.class, event -> {
    PlayerRef playerRef = event.getPlayerRef();
    getLogger().info("Player connecting: " + playerRef.getUsername());
});
```

### PlayerDisconnectEvent

Fired when a player disconnects.

**Extends:** `PlayerRefEvent<Void>`

| Property | Type | Description |
|----------|------|-------------|
| `playerRef` | `PlayerRef` | Player reference (inherited) |
| `disconnectReason` | `DisconnectReason` | Why the player disconnected |

```java
public class PlayerDisconnectEvent extends PlayerRefEvent<Void> {
    @Nonnull public PacketHandler.DisconnectReason getDisconnectReason();
}
```

### PlayerReadyEvent

Fired when a player is fully ready (loaded into world).

**Extends:** `PlayerEvent<String>`

| Property | Type | Description |
|----------|------|-------------|
| `ref` | `Ref<EntityStore>` | Entity reference (inherited) |
| `player` | `Player` | Player entity (inherited) |
| `readyId` | `int` | Ready identifier |

```java
public class PlayerReadyEvent extends PlayerEvent<String> {
    public int getReadyId();
}
```

### PlayerChatEvent

Fired when a player sends a chat message.

**Implements:** `IAsyncEvent<String>`, `ICancellable`

| Property | Type | Description |
|----------|------|-------------|
| `sender` | `PlayerRef` | Sending player |
| `targets` | `List<PlayerRef>` | Message recipients |
| `content` | `String` | Message content |
| `formatter` | `Formatter` | Message formatter |
| `cancelled` | `boolean` | Whether event is cancelled |

```java
public class PlayerChatEvent implements IAsyncEvent<String>, ICancellable {
    public static final Formatter DEFAULT_FORMATTER;
    
    @Nonnull public PlayerRef getSender();
    public void setSender(@Nonnull PlayerRef sender);
    
    @Nonnull public List<PlayerRef> getTargets();
    public void setTargets(@Nonnull List<PlayerRef> targets);
    
    @Nonnull public String getContent();
    public void setContent(@Nonnull String content);
    
    @Nonnull public Formatter getFormatter();
    public void setFormatter(@Nonnull Formatter formatter);
    
    public boolean isCancelled();
    public void setCancelled(boolean cancelled);
    
    public interface Formatter {
        @Nonnull Message format(@Nonnull PlayerRef playerRef, @Nonnull String message);
    }
}
```

```java
// PlayerChatEvent is async - use registerAsync
eventRegistry.registerAsync(PlayerChatEvent.class, future -> {
    return future.thenApply(event -> {
        if (event.getContent().contains("badword")) {
            event.setCancelled(true);
        }
        return event;
    });
});
```

### PlayerInteractEvent

Fired when a player interacts with something.

**Extends:** `PlayerEvent<String>`, **Implements:** `ICancellable`

**Note:** This event is marked `@Deprecated`

| Property | Type | Description |
|----------|------|-------------|
| `actionType` | `InteractionType` | Type of interaction |
| `clientUseTime` | `long` | Client-side use time |
| `itemInHand` | `ItemStack` | Item being used |
| `targetBlock` | `Vector3i` | Target block position |
| `targetRef` | `Ref<EntityStore>` | Target entity reference |
| `targetEntity` | `Entity` | Target entity |
| `cancelled` | `boolean` | Whether event is cancelled |

```java
@Deprecated
public class PlayerInteractEvent extends PlayerEvent<String> implements ICancellable {
    public InteractionType getActionType();
    public long getClientUseTime();
    public ItemStack getItemInHand();
    public Vector3i getTargetBlock();
    public Entity getTargetEntity();
    public Ref<EntityStore> getTargetRef();
    public boolean isCancelled();
    public void setCancelled(boolean cancelled);
}
```

### PlayerCraftEvent

Fired when a player crafts an item.

**Extends:** `PlayerEvent<String>`

**Note:** This event is marked `@Deprecated(forRemoval = true)`

| Property | Type | Description |
|----------|------|-------------|
| `craftedRecipe` | `CraftingRecipe` | The recipe used |
| `quantity` | `int` | Amount crafted |

### AddPlayerToWorldEvent

Fired when a player is added to a world.

**Implements:** `IEvent<String>`

| Property | Type | Description |
|----------|------|-------------|
| `holder` | `Holder<EntityStore>` | Entity holder |
| `world` | `World` | Target world |
| `broadcastJoinMessage` | `boolean` | Whether to broadcast join message |

```java
public class AddPlayerToWorldEvent implements IEvent<String> {
    @Nonnull public Holder<EntityStore> getHolder();
    @Nonnull public World getWorld();
    public boolean shouldBroadcastJoinMessage();
    public void setBroadcastJoinMessage(boolean broadcastJoinMessage);
}
```

### DrainPlayerFromWorldEvent

Fired when a player is removed from a world.

**Location:** `com.hypixel.hytale.server.core.event.events.player.DrainPlayerFromWorldEvent`

### PlayerSetupConnectEvent / PlayerSetupDisconnectEvent

Fired during player connection setup phase.

### PlayerMouseButtonEvent / PlayerMouseMotionEvent

Fired on mouse input events.

---

## Entity Events

Located in `com.hypixel.hytale.server.core.event.events.entity`:

### EntityRemoveEvent

Fired when an entity is removed from the world.

**Location:** `com.hypixel.hytale.server.core.event.events.entity.EntityRemoveEvent`

---

## ECS Events

Located in `com.hypixel.hytale.server.core.event.events.ecs`:

### ChangeGameModeEvent

Fired when a player's game mode changes.

---

## Asset Events

Located in `com.hypixel.hytale.assetstore.event`:

### AssetStoreEvent

Base class for asset store events.

### RegisterAssetStoreEvent

Fired when an asset store registers.

### RemoveAssetStoreEvent

Fired when an asset store is removed.

### GenerateAssetsEvent

Fired during asset generation.

### LoadedAssetsEvent

Fired after assets load.

### RemovedAssetsEvent

Fired after assets are removed.

### AssetMonitorEvent

Fired on asset file changes (hot-reload).

| Property | Type | Description |
|----------|------|-------------|
| `path` | `Path` | Changed file path |
| `eventKind` | `EventKind` | CREATE, MODIFY, or DELETE |

---

## LoadAssetEvent

Fired during the asset loading phase at boot.

**Location:** `com.hypixel.hytale.server.core.asset.LoadAssetEvent`

| Property | Type | Description |
|----------|------|-------------|
| `bootStart` | `Long` | Boot start timestamp |
| `shouldShutdown` | `Boolean` | Whether to abort boot |
| `reasons` | `List<String>` | Failure reasons |

```java
eventRegistry.register(LoadAssetEvent.class, event -> {
    if (!validateCustomAssets()) {
        event.setShouldShutdown(true);
        event.addReason("Custom asset validation failed");
    }
});
```

---

## Event Base Classes

### PlayerEvent<K>

Base class for player events with entity reference.

```java
public abstract class PlayerEvent<KeyType> implements IEvent<KeyType> {
    protected final Ref<EntityStore> ref;
    protected final Player player;
    
    @Nonnull public Ref<EntityStore> getRef();
    @Nonnull public Player getPlayer();
}
```

### PlayerRefEvent<K>

Base class for player events with PlayerRef.

```java
public abstract class PlayerRefEvent<KeyType> implements IEvent<KeyType> {
    protected final PlayerRef playerRef;
    
    @Nonnull public PlayerRef getPlayerRef();
}
```

---

## Event Interfaces

### IBaseEvent<K>

Base event interface with key type parameter.

### IEvent<K>

Synchronous event interface.

```java
public interface IEvent<KeyType> extends IBaseEvent<KeyType> {
    // Marker interface for sync events
}
```

### IAsyncEvent<K>

Asynchronous event interface.

```java
public interface IAsyncEvent<KeyType> extends IBaseEvent<KeyType> {
    // Marker interface for async events
}
```

### ICancellable

Interface for events that can be cancelled.

```java
public interface ICancellable {
    boolean isCancelled();
    void setCancelled(boolean cancelled);
}
```

---

## Event Priority

Events are handled in priority order:

```java
public enum EventPriority {
    LOWEST,    // First to execute
    LOW,
    NORMAL,    // Default
    HIGH,
    HIGHEST,
    MONITOR    // Last - observe only, do NOT modify
}
```

**Important:** `MONITOR` priority handlers should only observe events, never cancel or modify them.

---

## See Also

- [Event System](Event-System) - Event handling documentation
- [Creating Plugins](Creating-Plugins) - Event registration examples
- [Plugin System](Plugin-System) - Plugin EventRegistry access
