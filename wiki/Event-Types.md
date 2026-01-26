
This document provides a comprehensive reference of all event types in the Hytale Server. Events enable communication between systems and plugins.

## Overview

Events are dispatched through the [Event System](Event-System) to notify listeners of game occurrences. Events can be synchronous or asynchronous, and some can be cancelled.

## Event Categories

- [Server Events](#server-events)
- [Player Events](#player-events)
- [Entity Events](#entity-events)
- [World Events](#world-events)
- [Block Events](#block-events)
- [Inventory Events](#inventory-events)
- [Combat Events](#combat-events)
- [Asset Events](#asset-events)
- [Network Events](#network-events)

---

## Server Events

### BootEvent

Fired when the server has fully booted.

**Location:** `com.hypixel.hytale.server.core.event.events.BootEvent`

| Property | Type | Description |
|----------|------|-------------|
| - | - | No properties |

```java
eventBus.register(BootEvent.class, event -> {
    logger.info("Server is ready!");
});
```

### ShutdownEvent

Fired when the server begins shutdown.

**Location:** `com.hypixel.hytale.server.core.event.events.ShutdownEvent`

| Property | Type | Description |
|----------|------|-------------|
| - | - | No properties |

```java
eventBus.register(ShutdownEvent.class, event -> {
    saveAllData();
});
```

### LoadAssetEvent

Fired during asset loading phase.

**Location:** `com.hypixel.hytale.server.core.asset.LoadAssetEvent`

| Property | Type | Description |
|----------|------|-------------|
| `bootStart` | Long | Boot start timestamp |
| `shouldShutdown` | Boolean | Whether to abort |
| `reasons` | List<String> | Failure reasons |

```java
eventBus.register(LoadAssetEvent.class, event -> {
    if (validateAssets()) {
        // Assets OK
    } else {
        event.setShouldShutdown(true);
        event.addReason("Custom asset validation failed");
    }
});
```

---

## Player Events

### PlayerJoinEvent

Fired when a player joins the server.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Joining player |
| `world` | World | Spawn world |
| `position` | Vector3f | Spawn position |

```java
eventBus.register(PlayerJoinEvent.class, event -> {
    Player player = event.getPlayer();
    player.sendMessage("Welcome, " + player.getUsername() + "!");
});
```

### PlayerQuitEvent

Fired when a player leaves the server.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Leaving player |
| `reason` | QuitReason | Why they left |

```java
eventBus.register(PlayerQuitEvent.class, event -> {
    savePlayerData(event.getPlayer());
});
```

### PlayerChatEvent

Fired when a player sends a chat message.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Chatting player |
| `message` | String | Chat message |
| `cancelled` | Boolean | Cancel message |

**Cancellable:** Yes

```java
eventBus.register(PlayerChatEvent.class, event -> {
    // Filter bad words
    String filtered = filterMessage(event.getMessage());
    event.setMessage(filtered);

    // Or cancel entirely
    if (isMuted(event.getPlayer())) {
        event.setCancelled(true);
    }
});
```

### PlayerMoveEvent

Fired when a player moves.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Moving player |
| `from` | Location | Previous position |
| `to` | Location | New position |
| `cancelled` | Boolean | Cancel movement |

**Cancellable:** Yes

```java
eventBus.register(PlayerMoveEvent.class, event -> {
    if (isInRestrictedArea(event.getTo())) {
        event.setCancelled(true);
        event.getPlayer().sendMessage("You cannot enter this area!");
    }
});
```

### PlayerTeleportEvent

Fired when a player teleports.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Teleporting player |
| `from` | Location | Previous position |
| `to` | Location | Destination |
| `cause` | TeleportCause | Teleport reason |
| `cancelled` | Boolean | Cancel teleport |

**Cancellable:** Yes

### PlayerInteractEvent

Fired when a player interacts with something.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Interacting player |
| `action` | InteractAction | Interaction type |
| `hand` | Hand | Hand used |
| `item` | ItemStack | Item in hand |
| `cancelled` | Boolean | Cancel interaction |

**Cancellable:** Yes

### PlayerRespawnEvent

Fired when a player respawns.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Respawning player |
| `respawnLocation` | Location | Spawn location |
| `isBedSpawn` | Boolean | Spawning at bed |

---

## Entity Events

### EntitySpawnEvent

Fired when an entity spawns.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Spawned entity |
| `location` | Location | Spawn location |
| `reason` | SpawnReason | Why it spawned |
| `cancelled` | Boolean | Cancel spawn |

**Cancellable:** Yes

```java
eventBus.register(EntitySpawnEvent.class, event -> {
    if (event.getEntity().getType().equals("zombie")) {
        if (isSpawnDisabled(event.getLocation().getWorld())) {
            event.setCancelled(true);
        }
    }
});
```

### EntityDespawnEvent

Fired when an entity despawns.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Despawning entity |
| `reason` | DespawnReason | Why it despawned |

### EntityDamageEvent

Fired when an entity takes damage.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Damaged entity |
| `damage` | Float | Damage amount |
| `source` | DamageSource | Damage source |
| `attacker` | Entity | Attacking entity |
| `cancelled` | Boolean | Cancel damage |

**Cancellable:** Yes

```java
eventBus.register(EntityDamageEvent.class, event -> {
    if (event.getEntity() instanceof Player) {
        Player player = (Player) event.getEntity();
        if (isProtected(player)) {
            event.setCancelled(true);
        }
    }
});
```

### EntityDeathEvent

Fired when an entity dies.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Dying entity |
| `killer` | Entity | Killing entity |
| `drops` | List<ItemStack> | Dropped items |
| `experience` | Integer | XP dropped |

```java
eventBus.register(EntityDeathEvent.class, event -> {
    // Modify drops
    event.getDrops().add(new ItemStack("bonus_item", 1));

    // Modify experience
    event.setExperience(event.getExperience() * 2);
});
```

### EntityHealEvent

Fired when an entity heals.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Healing entity |
| `amount` | Float | Heal amount |
| `reason` | HealReason | Why healing |
| `cancelled` | Boolean | Cancel healing |

**Cancellable:** Yes

### EntityTargetEvent

Fired when an entity targets another.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Targeting entity |
| `target` | Entity | New target |
| `reason` | TargetReason | Why targeting |
| `cancelled` | Boolean | Cancel targeting |

**Cancellable:** Yes

---

## World Events

### WorldLoadEvent

Fired when a world loads.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Loaded world |

### WorldUnloadEvent

Fired when a world unloads.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Unloading world |
| `cancelled` | Boolean | Cancel unload |

**Cancellable:** Yes

### WorldSaveEvent

Fired when a world saves.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Saving world |

### ChunkLoadEvent

Fired when a chunk loads.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Chunk world |
| `chunk` | ChunkColumn | Loaded chunk |
| `isNew` | Boolean | Newly generated |

### ChunkUnloadEvent

Fired when a chunk unloads.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Chunk world |
| `chunk` | ChunkColumn | Unloading chunk |
| `save` | Boolean | Should save |
| `cancelled` | Boolean | Cancel unload |

**Cancellable:** Yes

### ChunkGenerateEvent

Fired when a chunk is generated.

| Property | Type | Description |
|----------|------|-------------|
| `world` | World | Chunk world |
| `chunkX` | Integer | Chunk X coordinate |
| `chunkZ` | Integer | Chunk Z coordinate |

---

## Block Events

### BlockBreakEvent

Fired when a block is broken.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Breaking player |
| `block` | Block | Broken block |
| `drops` | List<ItemStack> | Block drops |
| `experience` | Integer | XP dropped |
| `cancelled` | Boolean | Cancel break |

**Cancellable:** Yes

```java
eventBus.register(BlockBreakEvent.class, event -> {
    BlockType type = event.getBlock().getType();

    // Prevent breaking certain blocks
    if (type.getName().equals("bedrock")) {
        event.setCancelled(true);
        event.getPlayer().sendMessage("Cannot break bedrock!");
    }

    // Modify drops
    if (type.getName().equals("diamond_ore")) {
        event.getDrops().add(new ItemStack("bonus_diamond", 1));
    }
});
```

### BlockPlaceEvent

Fired when a block is placed.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Placing player |
| `block` | Block | Placed block |
| `placedAgainst` | Block | Adjacent block |
| `itemInHand` | ItemStack | Item used |
| `cancelled` | Boolean | Cancel placement |

**Cancellable:** Yes

### BlockInteractEvent

Fired when a player interacts with a block.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Interacting player |
| `block` | Block | Target block |
| `action` | Action | Interaction type |
| `face` | BlockFace | Clicked face |
| `cancelled` | Boolean | Cancel interaction |

**Cancellable:** Yes

### BlockUpdateEvent

Fired when a block updates.

| Property | Type | Description |
|----------|------|-------------|
| `block` | Block | Updating block |
| `cause` | UpdateCause | Update reason |

### BlockPhysicsEvent

Fired for block physics updates.

| Property | Type | Description |
|----------|------|-------------|
| `block` | Block | Affected block |
| `sourceBlock` | Block | Cause block |
| `cancelled` | Boolean | Cancel physics |

**Cancellable:** Yes

### BlockGrowEvent

Fired when a block grows (crops, etc.).

| Property | Type | Description |
|----------|------|-------------|
| `block` | Block | Growing block |
| `newState` | BlockState | New state |
| `cancelled` | Boolean | Cancel growth |

**Cancellable:** Yes

### BlockSpreadEvent

Fired when a block spreads (fire, grass, etc.).

| Property | Type | Description |
|----------|------|-------------|
| `source` | Block | Source block |
| `block` | Block | Spreading to |
| `newState` | BlockState | New state |
| `cancelled` | Boolean | Cancel spread |

**Cancellable:** Yes

---

## Inventory Events

### InventoryOpenEvent

Fired when an inventory opens.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Opening player |
| `inventory` | Inventory | Opened inventory |
| `cancelled` | Boolean | Cancel open |

**Cancellable:** Yes

### InventoryCloseEvent

Fired when an inventory closes.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Closing player |
| `inventory` | Inventory | Closed inventory |

### InventoryClickEvent

Fired when clicking in an inventory.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Clicking player |
| `inventory` | Inventory | Target inventory |
| `slot` | Integer | Clicked slot |
| `clickType` | ClickType | Click type |
| `item` | ItemStack | Clicked item |
| `cancelled` | Boolean | Cancel click |

**Cancellable:** Yes

### ItemPickupEvent

Fired when picking up an item.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Picking up player |
| `item` | ItemEntity | Picked item |
| `remaining` | Integer | Leftover count |
| `cancelled` | Boolean | Cancel pickup |

**Cancellable:** Yes

### ItemDropEvent

Fired when dropping an item.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Dropping player |
| `item` | ItemStack | Dropped item |
| `cancelled` | Boolean | Cancel drop |

**Cancellable:** Yes

### CraftItemEvent

Fired when crafting an item.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Crafting player |
| `recipe` | Recipe | Used recipe |
| `result` | ItemStack | Crafted item |
| `cancelled` | Boolean | Cancel craft |

**Cancellable:** Yes

---

## Combat Events

### PlayerAttackEvent

Fired when a player attacks.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Attacking player |
| `target` | Entity | Attack target |
| `damage` | Float | Damage dealt |
| `cancelled` | Boolean | Cancel attack |

**Cancellable:** Yes

### ProjectileLaunchEvent

Fired when a projectile launches.

| Property | Type | Description |
|----------|------|-------------|
| `entity` | Entity | Shooter entity |
| `projectile` | Entity | Projectile |
| `cancelled` | Boolean | Cancel launch |

**Cancellable:** Yes

### ProjectileHitEvent

Fired when a projectile hits.

| Property | Type | Description |
|----------|------|-------------|
| `projectile` | Entity | Projectile |
| `hitEntity` | Entity | Hit entity |
| `hitBlock` | Block | Hit block |

### ExplosionEvent

Fired when an explosion occurs.

| Property | Type | Description |
|----------|------|-------------|
| `location` | Location | Explosion center |
| `radius` | Float | Explosion radius |
| `blocks` | List<Block> | Affected blocks |
| `entities` | List<Entity> | Affected entities |
| `cancelled` | Boolean | Cancel explosion |

**Cancellable:** Yes

---

## Asset Events

### AssetStoreEvent

Base class for asset store events.

**Location:** `com.hypixel.hytale.assetstore.event.AssetStoreEvent`

### RegisterAssetStoreEvent

Fired when an asset store registers.

| Property | Type | Description |
|----------|------|-------------|
| `assetStore` | AssetStore | Registered store |

### RemoveAssetStoreEvent

Fired when an asset store is removed.

| Property | Type | Description |
|----------|------|-------------|
| `assetStore` | AssetStore | Removed store |

### GenerateAssetsEvent

Fired during asset generation.

| Property | Type | Description |
|----------|------|-------------|
| `parentReference` | ParentReference | Parent asset |

### LoadedAssetsEvent

Fired after assets load.

| Property | Type | Description |
|----------|------|-------------|
| `assetStore` | AssetStore | Loaded store |
| `assets` | Collection | Loaded assets |

### RemovedAssetsEvent

Fired after assets are removed.

| Property | Type | Description |
|----------|------|-------------|
| `assetStore` | AssetStore | Store |
| `assets` | Collection | Removed assets |

### AssetMonitorEvent

Fired on asset file changes (hot-reload).

**Location:** `com.hypixel.hytale.assetstore.event.AssetMonitorEvent`

| Property | Type | Description |
|----------|------|-------------|
| `path` | Path | Changed file |
| `eventKind` | EventKind | CREATE/MODIFY/DELETE |

---

## Network Events

### PacketReceiveEvent

Fired when receiving a packet.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Source player |
| `packet` | Packet | Received packet |
| `cancelled` | Boolean | Cancel packet |

**Cancellable:** Yes

### PacketSendEvent

Fired when sending a packet.

| Property | Type | Description |
|----------|------|-------------|
| `player` | Player | Target player |
| `packet` | Packet | Sent packet |
| `cancelled` | Boolean | Cancel packet |

**Cancellable:** Yes

### PlayerConnectionEvent

Fired during connection process.

| Property | Type | Description |
|----------|------|-------------|
| `address` | InetAddress | Client address |
| `state` | ConnectionState | Current state |

---

## Event Interfaces

### IEvent<K>

Synchronous event interface.

```java
public interface IEvent<KeyType> extends IBaseEvent<KeyType> {
    // Marker interface
}
```

### IAsyncEvent<K>

Asynchronous event interface.

```java
public interface IAsyncEvent<KeyType> extends IBaseEvent<KeyType> {
    // Marker interface
}
```

### ICancellable

Cancellable event interface.

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
    LOWEST((short)-200),   // First to execute
    LOW((short)-100),
    NORMAL((short)0),      // Default
    HIGH((short)100),
    HIGHEST((short)200),
    MONITOR((short)300)    // Last, observe only
}
```

**Important:** MONITOR priority should only observe, never modify events.

---

## See Also

- [Event System](Event-System) - Event handling documentation
- [Creating Plugins](Creating-Plugins) - Event registration examples
- [Example Plugins](Example-Plugins) - Event listener examples
