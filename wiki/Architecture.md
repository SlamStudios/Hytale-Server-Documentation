
This document describes the high-level architecture of the Hytale Server.

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           HytaleServer                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌───────────────┐  ┌──────────────────────────┐  │
│  │   EventBus   │  │ PluginManager │  │      CommandManager      │  │
│  └──────────────┘  └───────────────┘  └──────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                        Universe                               │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────────────────┐ │  │
│  │  │ World 1 │  │ World 2 │  │ World N │  │ DataStoreProvider │ │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └───────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     ServerManager (I/O)                      │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  │   │
│  │  │ QUICTransport  │  │  TCPTransport  │  │ PacketHandler  │  │   │
│  │  └────────────────┘  └────────────────┘  └────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                      Asset System                            │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────────────────┐  │   │
│  │  │ AssetStore │  │ AssetPack  │  │ AssetMonitor (hot-load)│  │   │
│  │  └────────────┘  └────────────┘  └────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## Core Classes

### HytaleServer (`com.hypixel.hytale.server.core.HytaleServer`)

The main server class that orchestrates all subsystems.

**Key Fields:**
- `eventBus` - Central event dispatching system
- `pluginManager` - Manages plugin lifecycle
- `commandManager` - Handles console and player commands
- `hytaleServerConfig` - Server configuration

**Key Methods:**
- `boot()` - Initializes and starts the server
- `shutdownServer(ShutdownReason)` - Gracefully shuts down the server
- `getConfig()` - Returns server configuration

### Entry Points

1. **Main.java** - Application entry point
    - Sets locale and system properties
    - Loads early plugins (bytecode transformers)
    - Launches `TransformingClassLoader` if transformers exist
    - Calls `LateMain.lateMain()`

2. **LateMain.java** - Post-classloader initialization
    - Parses command-line options
    - Initializes logging system
    - Creates `HytaleServer` instance

## Subsystems

### 1. Plugin System

The server uses a modular plugin architecture:

```
PluginManager
├── CorePlugins (built-in)
├── JavaPlugins (external JARs)
└── PluginClassLoader
```

**Plugin Lifecycle:**
1. `NONE` - Initial state
2. `SETUP` - Plugin configuration
3. `START` - Plugin activation
4. `ENABLED` - Running state
5. `SHUTDOWN` - Stopping
6. `DISABLED` - Stopped state

### 2. Event System

Asynchronous and synchronous event bus:

```java
// Registering an event handler
eventBus.register(SomeEvent.class, event -> {
    // Handle event
});

// Dispatching an event
eventBus.dispatch(SomeEvent.class);
```

**Event Types:**
- `IEvent<K>` - Synchronous events
- `IAsyncEvent<K>` - Asynchronous events
- `ICancellable` - Cancellable events

### 3. Component System (ECS)

Entity Component System architecture:

```
ComponentRegistry
├── Component (data)
├── System (behavior)
├── Archetype (entity template)
└── Query (entity selection)
```

**Key Concepts:**
- **Component**: Pure data, no behavior
- **System**: Processes entities with specific components
- **Archetype**: Template defining entity component composition
- **Query**: Selects entities based on component criteria

### 4. Universe & World

The game world hierarchy:

```
Universe
├── World (dimension/realm)
│   ├── ChunkColumn
│   │   ├── WorldChunk
│   │   ├── BlockChunk
│   │   ├── EntityChunk
│   │   └── BlockComponentChunk
│   └── WorldConfig
└── DataStoreProvider
```

### 5. Networking

Uses Netty for network I/O:

```
ServerManager
├── Transport
│   ├── QUICTransport (primary)
│   └── TCPTransport (fallback)
├── PacketHandler
│   ├── HandshakeHandler
│   ├── AuthenticationPacketHandler
│   ├── GamePacketHandler
│   └── InventoryPacketHandler
└── HytaleChannelInitializer
```

## Configuration

Server configuration is stored in `HytaleServerConfig`:

| Setting | Description |
|---------|-------------|
| `serverName` | Server display name |
| `maxPlayers` | Maximum concurrent players |
| `logLevels` | Per-module log levels |
| `modules` | Module-specific configuration |
| `rateLimits` | Rate limiting settings |
| `timeoutProfile` | Connection timeout settings |

## Thread Model

The server uses multiple thread pools:

1. **Main Thread** - Server tick loop
2. **Netty Event Loops** - Network I/O
3. **Scheduled Executor** - Timed tasks
4. **ForkJoin Pool** - Parallel processing
5. **World Threads** - Per-world tick processing

## Boot Sequence

1. Initialize locale and system properties
2. Load early plugins (transformers)
3. Initialize logging system
4. Load server configuration
5. Initialize authentication
6. Initialize Netty networking
7. Register core plugins
8. Load assets (`LoadAssetEvent`)
9. Setup plugins
10. Start plugins
11. Initialize Universe
12. Wait for bind completion
13. Dispatch `BootEvent`
14. Execute boot commands

## Shutdown Sequence

1. Receive shutdown signal
2. Dispatch `ShutdownEvent`
3. Shutdown plugins (reverse order)
4. Shutdown command manager
5. Shutdown event bus
6. Shutdown authentication
7. Save configuration
8. Release alive lock
9. Exit with status code

## Error Handling

The server uses Sentry for error reporting (when enabled):

- Captures unhandled exceptions
- Includes context (plugins, universe state)
- Filters third-party plugin exceptions
- Respects privacy settings
