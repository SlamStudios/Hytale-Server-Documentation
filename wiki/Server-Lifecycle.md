
This document describes the complete lifecycle of a Hytale Server from startup to shutdown.

## Overview

The server lifecycle consists of:
1. Startup phase
2. Boot phase
3. Running phase
4. Shutdown phase

## Startup Phase

### Entry Point

```
Main.main(String[] args)
├── Set locale (English)
├── Set system properties
│   ├── java.awt.headless = true
│   └── file.encoding = UTF-8
├── Load early plugins
│   └── Bytecode transformers
└── Launch LateMain
    ├── Direct (no transformers)
    └── Via TransformingClassLoader
```

### Early Plugins

Early plugins can modify classes before loading:

```java
EarlyPluginLoader.loadEarlyPlugins(args);

if (EarlyPluginLoader.hasTransformers()) {
    // Launch with transforming classloader
    launchWithTransformingClassLoader(args);
} else {
    // Direct launch
    LateMain.lateMain(args);
}
```

### TransformingClassLoader

Allows bytecode transformation:

```java
TransformingClassLoader classLoader = new TransformingClassLoader(
    urls,
    transformers,
    parentClassLoader,
    appClassLoader
);
```

## Late Initialization

```
LateMain.lateMain(String[] args)
├── Parse options
├── Initialize logging
│   ├── HytaleLogger.init()
│   ├── HytaleFileHandler.enable()
│   └── HytaleLogger.replaceStd()
├── Configure log levels
└── Create HytaleServer
```

### Option Parsing

```java
if (Options.parse(args)) {
    return; // Help displayed, exit
}
```

## Boot Phase

### HytaleServer Constructor

```java
HytaleServer()
├── Ensure QUIC availability
├── Configure logger indent
├── Force high-resolution time
├── Create keep-alive thread
├── Record boot time
├── Log "Starting HytaleServer"
├── Initialize Constants
├── Register DataStore codecs
├── Load configuration
├── Reload log levels
├── Configure ForkJoinPool
├── Log authentication mode
├── Initialize authentication
├── Initialize Sentry (optional)
├── Check auth errors
├── Initialize Netty
├── Warm up trigonometry
├── Register shutdown hook
├── Initialize AssetRegistry
├── Register core plugins
├── Register GC listener
└── Call boot()
```

### Boot Method

```java
boot()
├── Mark booting started
├── Log version info
├── Reset progress
├── Setup phase
│   ├── Register commands
│   ├── Setup plugins
│   └── Initialize credential store
├── Load assets
│   ├── Dispatch LoadAssetEvent
│   ├── Validate assets
│   └── Handle validation errors
├── Start phase
│   └── Start all plugins
├── Initialize Universe
│   └── Wait for ready
├── Complete boot
│   ├── Wait for bind
│   ├── Dispatch BootEvent
│   └── Execute boot commands
└── Log "Booted!"
```

## Plugin Lifecycle

### Plugin States

```
NONE → SETUP → START → ENABLED
                         ↓
              SHUTDOWN → DISABLED
```

### Setup Phase

```java
pluginManager.setup()
├── Sort plugins by dependency
├── For each plugin:
│   ├── Call preLoad()
│   ├── Wait for config load
│   ├── Call setup0()
│   │   └── Plugin.setup()
│   └── Report progress
└── Log completion
```

### Start Phase

```java
pluginManager.start()
├── For each plugin:
│   ├── Call start0()
│   │   └── Plugin.start()
│   └── Report progress
└── Log completion
```

## Running Phase

### Main Loop

The server runs until shutdown:

```java
// Keep-alive lock holds the JVM
aliveLock.acquire();
```

### Scheduled Tasks

```java
SCHEDULED_EXECUTOR.scheduleWithFixedDelay(
    () -> saveConfigIfChanged(),
    1, 1, TimeUnit.MINUTES
);
```

### World Ticking

Worlds tick independently with their own thread pools.

## Shutdown Phase

### Shutdown Triggers

```java
// Programmatic shutdown
server.shutdownServer();
server.shutdownServer(ShutdownReason.CUSTOM);

// SIGINT (Ctrl+C)
// Triggers shutdown hook

// Fatal error
// With ShutdownReason.CRASH
```

### Shutdown Reasons

```java
public enum ShutdownReason {
    SHUTDOWN,        // Normal shutdown
    SIGINT,          // Interrupt signal
    CRASH,           // Unrecoverable error
    VALIDATE_ERROR,  // Asset validation failed
    // Custom with message
}
```

### Shutdown Sequence

```java
shutdown0(ShutdownReason reason)
├── Log "Shutdown triggered"
├── Log reason and code
├── Dispatch ShutdownEvent
├── Shutdown plugins (reverse order)
│   ├── Call shutdown0() for each
│   └── Call cleanup()
├── Shutdown command manager
├── Shutdown event bus
├── Shutdown authentication
├── Save configuration
├── Log "Shutdown completed"
├── Release alive lock
├── Reset log manager
├── Schedule force exit (3s)
└── System.exit(code)
```

### Plugin Shutdown

```java
plugin.shutdown0(shutdown)
├── Set state to SHUTDOWN
├── Call shutdown()
├── Set state to DISABLED
└── Call cleanup()
    ├── Shutdown command registry
    ├── Shutdown event registry
    ├── Shutdown client features
    ├── Shutdown block states
    ├── Shutdown tasks
    ├── Shutdown stores
    ├── Shutdown codecs
    ├── Shutdown assets
    └── Execute shutdown tasks
```

## Singleplayer Mode

### Progress Reporting

```java
// During startup
sendSingleplayerSignal("-=|Setup|" + progress);
sendSingleplayerSignal("-=|Starting|" + progress);
sendSingleplayerSignal("-=|Enabled|0");

// During shutdown
sendSingleplayerSignal("-=|Shutdown|message");
sendSingleplayerSignal("-=|Shutdown Modules|" + progress);
sendSingleplayerSignal("-=|Saving world X chunks|" + progress);

// Ready signal
sendSingleplayerSignal(">> Singleplayer Ready <<");
```

## Error Handling

### Boot Failure

```java
try {
    // Boot logic
} catch (Throwable t) {
    logger.severe("Failed to boot HytaleServer!", t);
    shutdownServer(ShutdownReason.CRASH.withMessage(message));
}
```

### Sentry Integration

```java
if (!SkipSentryException.hasSkipSentry(t)) {
    Sentry.captureException(t);
}
```

## Exit Codes

| Code | Reason |
|------|--------|
| 0 | Normal shutdown |
| 1 | Crash/error |
| 2 | Validation error |
| Other | Custom reason |

## Events During Lifecycle

| Phase | Events |
|-------|--------|
| Boot | `LoadAssetEvent`, `BootEvent` |
| Shutdown | `ShutdownEvent` |
| Asset Changes | `AssetMonitorEvent` |

## Thread Management

### Alive Lock

```java
Semaphore aliveLock = new Semaphore(0);

// During boot
ThreadUtil.createKeepAliveThread(aliveLock);

// JVM stays alive until
aliveLock.release();
```

### Daemon Threads

Most threads are daemon threads, allowing clean shutdown:

```java
Thread thread = ThreadUtil.daemon("ThreadName");
```

## GC Notifications

```java
GCUtil.register(info -> {
    // Notify worlds of GC
    for (World world : universe.getWorlds().values()) {
        world.markGCHasRun();
    }
});
```

## Best Practices

1. **Handle shutdown gracefully** - Save state
2. **Use plugin lifecycle** - setup/start/shutdown
3. **Register cleanup tasks** - Via shutdown tasks
4. **Check isShuttingDown()** - Abort long operations
5. **Use appropriate shutdown reason** - For debugging
6. **Don't block shutdown** - Keep shutdown fast
