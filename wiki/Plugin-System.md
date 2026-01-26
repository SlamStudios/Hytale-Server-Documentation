
The Hytale Server uses a modular plugin system that allows features to be loaded dynamically. This document covers the plugin architecture, lifecycle, and how to create plugins.

## Overview

The plugin system is managed by `PluginManager` and supports two types of plugins:

1. **Core Plugins** - Built-in plugins bundled with the server
2. **Java Plugins** - External plugins loaded from JAR files

## Plugin Architecture

```
PluginManager
├── Core Plugins (built-in)
│   └── PluginBase implementations
├── Java Plugins (external)
│   └── Loaded via PluginClassLoader
└── PluginBridgeClassLoader
    └── Provides class isolation
```

### Key Classes

| Class | Location | Description |
|-------|----------|-------------|
| `PluginManager` | `server.core.plugin` | Manages plugin lifecycle |
| `PluginBase` | `server.core.plugin` | Base class for all plugins |
| `JavaPlugin` | `server.core.plugin` | External plugin base class |
| `PluginClassLoader` | `server.core.plugin` | Class isolation for plugins |
| `PluginManifest` | `common.plugin` | Plugin metadata descriptor |
| `PluginIdentifier` | `common.plugin` | Unique plugin identifier |

## Plugin Lifecycle

```
NONE → SETUP → START → ENABLED
                         ↓
              SHUTDOWN → DISABLED
```

### States

| State | Description |
|-------|-------------|
| `NONE` | Initial state, not yet loaded |
| `SETUP` | Plugin is being configured |
| `START` | Plugin is starting |
| `ENABLED` | Plugin is running |
| `SHUTDOWN` | Plugin is shutting down |
| `DISABLED` | Plugin is disabled/stopped |

### Lifecycle Methods

```java
public abstract class PluginBase {
    // Called during SETUP phase
    protected void setup() {}

    // Called during START phase
    protected void start() {}

    // Called during SHUTDOWN phase
    protected void shutdown() {}
}
```

## PluginBase Class

The `PluginBase` class provides the foundation for all plugins:

### Available Registries

| Registry | Method | Purpose |
|----------|--------|---------|
| Command Registry | `getCommandRegistry()` | Register commands |
| Event Registry | `getEventRegistry()` | Register event handlers |
| Entity Registry | `getEntityRegistry()` | Register entity types |
| Block State Registry | `getBlockStateRegistry()` | Register block states |
| Task Registry | `getTaskRegistry()` | Schedule tasks |
| Asset Registry | `getAssetRegistry()` | Register asset types |
| Client Feature Registry | `getClientFeatureRegistry()` | Client-side features |
| Entity Store Registry | `getEntityStoreRegistry()` | Entity persistence |
| Chunk Store Registry | `getChunkStoreRegistry()` | Chunk persistence |

### Configuration

Plugins can define configuration files:

```java
public class MyPlugin extends JavaPlugin {
    private Config<MyConfig> config;

    public MyPlugin(JavaPluginInit init) {
        super(init);
        // Register config before setup
        config = withConfig("config", MyConfig.CODEC);
    }

    @Override
    protected void setup() {
        MyConfig cfg = config.get();
        // Use configuration
    }
}
```

### Logging

Each plugin has its own logger:

```java
HytaleLogger logger = getLogger();
logger.at(Level.INFO).log("Plugin loaded");
logger.at(Level.WARNING).log("Warning message");
logger.at(Level.SEVERE).withCause(exception).log("Error occurred");
```

## Plugin Manifest

The `PluginManifest` contains plugin metadata:

```json
{
    "name": "MyPlugin",
    "group": "com.example",
    "version": "1.0.0",
    "main": "com.example.myplugin.MyPlugin",
    "description": "My awesome plugin",
    "authors": ["Author Name"],
    "dependencies": ["OtherPlugin"]
}
```

### Manifest Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin name |
| `group` | Yes | Plugin group/namespace |
| `version` | Yes | Semantic version |
| `main` | Yes | Main class path |
| `description` | No | Plugin description |
| `authors` | No | List of authors |
| `dependencies` | No | Required plugins |

## Built-in Plugins

The server includes numerous built-in plugins:

### Adventure Plugins
- `CameraPlugin` - Camera effects and shake
- `FarmingPlugin` - Crop farming system
- `MemoriesPlugin` - NPC memory system
- `ObjectivePlugin` - Quest objectives
- `ReputationPlugin` - Reputation system
- `ShopPlugin` - Trading/shop system
- `StashPlugin` - Item storage
- `TeleporterPlugin` - Teleportation

### Core Plugins
- `AmbiencePlugin` - Ambient sounds/effects
- `AssetEditorPlugin` - In-game asset editing
- `BedsPlugin` - Bed mechanics
- `BlockPhysicsPlugin` - Block physics simulation
- `BlockSpawnerPlugin` - Mob spawners
- `BlockTickPlugin` - Block tick scheduling
- `BuilderToolsPlugin` - World editing tools
- `CraftingPlugin` - Crafting system
- `CreativeHubPlugin` - Creative mode hub
- `CrouchSlidePlugin` - Movement mechanics
- `DeployablesPlugin` - Placeable items
- `FluidPlugin` - Water/lava simulation
- `InstancesPlugin` - Dungeon instances
- `LANDiscoveryPlugin` - LAN server discovery
- `MantlingPlugin` - Ledge climbing
- `ModelPlugin` - Entity models
- `MountPlugin` - Mount system
- `NPCEditorPlugin` - NPC editing tools
- `NPCCombatActionEvaluatorPlugin` - NPC combat AI
- `ParkourPlugin` - Parkour mechanics
- `PathPlugin` - Pathfinding
- `PortalsPlugin` - Portal system
- `SafetyRollPlugin` - Fall damage reduction
- `SprintForcePlugin` - Sprint mechanics
- `TagSetPlugin` - Entity tags
- `TeleportPlugin` - Teleportation commands
- `WeatherPlugin` - Weather system
- `WorldGenPlugin` - World generation

## Plugin Types

### PluginType Enum

```java
public enum PluginType {
    CORE,    // Built-in core plugin
    PLUGIN   // External Java plugin
}
```

## Event Registration

Plugins register event handlers through their event registry:

```java
@Override
protected void setup() {
    getEventRegistry().register(PlayerJoinEvent.class, this::onPlayerJoin);
    getEventRegistry().register(EventPriority.HIGH, BlockBreakEvent.class, this::onBlockBreak);
}

private void onPlayerJoin(PlayerJoinEvent event) {
    // Handle event
}

private void onBlockBreak(BlockBreakEvent event) {
    if (shouldCancel(event)) {
        event.setCancelled(true);
    }
}
```

## Command Registration

Plugins register commands through their command registry:

```java
@Override
protected void setup() {
    getCommandRegistry().register(new MyCommand());
}
```

## Permissions

Plugins define a base permission automatically:

```java
// Base permission: "{group}.{name}"
String basePermission = getBasePermission();
// e.g., "com.example.myplugin"
```

## Data Directory

Each plugin has a dedicated data directory:

```java
Path dataDir = getDataDirectory();
// Used for configs, data files, etc.
```

## Plugin Dependencies

Plugins can depend on other plugins:

```java
// In manifest
{
    "dependencies": ["RequiredPlugin", "AnotherPlugin"]
}
```

The `PluginManager` ensures dependencies are loaded before dependent plugins.

## Third-Party Plugin Detection

The server tracks whether exceptions originate from third-party plugins:

```java
// PluginClassLoader.isFromThirdPartyPlugin(throwable)
// Used to filter Sentry error reports
```

## Best Practices

1. **Register resources in setup()** - Commands, events, etc.
2. **Start background tasks in start()** - After dependencies are ready
3. **Clean up in shutdown()** - Stop tasks, save data
4. **Use the event registry** - Automatically cleaned up on disable
5. **Check plugin state** - Before accessing other plugins
6. **Handle exceptions** - Don't crash the server
7. **Log appropriately** - Use the plugin logger
