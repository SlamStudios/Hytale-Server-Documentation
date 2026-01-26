
The Hytale Server uses JSON-based configuration for server settings and module configuration. This document covers all configuration options.

## Overview

Configuration is managed through:
- `HytaleServerConfig` - Main server configuration
- `Options` - Command-line options
- Plugin configs - Per-plugin configuration

## Server Configuration

### Location

Configuration is typically stored in the server directory as `server-config.json`.

### HytaleServerConfig

Main configuration class:

```java
HytaleServerConfig config = HytaleServer.get().getConfig();

// Access settings
String serverName = config.getServerName();
int maxPlayers = config.getMaxPlayers();
Map<String, Level> logLevels = config.getLogLevels();
```

## Configuration Options

### Basic Settings

| Setting | Type | Description |
|---------|------|-------------|
| `serverName` | String | Server display name |
| `maxPlayers` | Integer | Maximum concurrent players |

### Network Settings

| Setting | Type | Description |
|---------|------|-------------|
| `port` | Integer | Server port (default: 5520) |
| `address` | String | Bind address |

### Module Configuration

```json
{
    "modules": {
        "moduleName": {
            "enabled": true,
            "setting1": "value1",
            "setting2": 100
        }
    }
}
```

### Rate Limiting

```java
public class RateLimitConfig {
    int packetsPerSecond;
    int bytesPerSecond;
}
```

```json
{
    "rateLimits": {
        "packetsPerSecond": 1000,
        "bytesPerSecond": 1000000
    }
}
```

### Timeout Profile

```java
public class TimeoutProfile {
    // Connection timeouts
    // Read/write timeouts
    // Keep-alive settings
}
```

### Update Configuration

```java
public class UpdateConfig {
    enum AutoApplyMode {
        NEVER,
        RESTART,
        ALWAYS
    }
}
```

### Log Levels

Per-module logging configuration:

```json
{
    "logLevels": {
        "com.hypixel.hytale.server": "INFO",
        "com.hypixel.hytale.network": "WARNING",
        "myplugin": "FINE"
    }
}
```

Available levels:
- `SEVERE`
- `WARNING`
- `INFO`
- `CONFIG`
- `FINE`
- `FINER`
- `FINEST`

### Defaults

```java
public class Defaults {
    // Default configuration values
}
```

## Command-Line Options

### Options Class

Command-line argument parsing:

```java
Options.parse(args);
OptionSet optionSet = Options.getOptionSet();
```

### Available Options

| Option | Description |
|--------|-------------|
| `--port <port>` | Server port |
| `--auth-mode <mode>` | Authentication mode |
| `--log-level <name>=<level>` | Set log level |
| `--boot-command <cmd>` | Execute on boot |
| `--disable-sentry` | Disable error reporting |
| `--shutdown-after-validate` | Validate and exit |
| `--event-debug` | Enable event timing |
| `--universe <path>` | Universe directory |

### Authentication Modes

```java
public enum AuthMode {
    NONE,           // No authentication
    ONLINE,         // Online authentication
    SINGLEPLAYER    // Singleplayer mode
}
```

### Path Options

```java
Options.PathConverter
├── PathType.FILE
├── PathType.DIRECTORY
└── PathType.ANY
```

## Plugin Configuration

### Creating Plugin Config

```java
public class MyPlugin extends JavaPlugin {
    private Config<MyConfig> config;

    public MyPlugin(JavaPluginInit init) {
        super(init);
        // Must be called before setup()
        config = withConfig("config", MyConfig.CODEC);
    }

    @Override
    protected void setup() {
        MyConfig cfg = config.get();
        // Use config
    }
}
```

### Config Class

```java
public class Config<T> {
    // Load configuration
    CompletableFuture<Void> load();

    // Get config value
    T get();

    // Save configuration
    CompletableFuture<Void> save();
}
```

### Defining Config Schema

Using codecs:

```java
public class MyConfig {
    public static final BuilderCodec<MyConfig> CODEC = BuilderCodec.builder(MyConfig.class)
        .with("setting1", Codec.STRING, MyConfig::getSetting1, MyConfig.Builder::setting1)
        .with("setting2", Codec.INT, MyConfig::getSetting2, MyConfig.Builder::setting2)
        .with("enabled", Codec.BOOLEAN, MyConfig::isEnabled, MyConfig.Builder::enabled)
        .build();

    private final String setting1;
    private final int setting2;
    private final boolean enabled;

    // Constructor, getters, builder...
}
```

### Config File Location

Plugin configs are stored in the plugin's data directory:

```
plugins/
└── MyPlugin/
    └── config.json
```

### Named Configs

```java
// Default config
Config<MainConfig> mainConfig = withConfig("config", MainConfig.CODEC);

// Named config
Config<OtherConfig> otherConfig = withConfig("other", OtherConfig.CODEC);
// Stored as: plugins/MyPlugin/other.json
```

## Constants

Server constants:

```java
Constants.init();

// Available constants
boolean SINGLEPLAYER = Constants.SINGLEPLAYER;
boolean FRESH_UNIVERSE = Constants.FRESH_UNIVERSE;
List<PluginManifest> CORE_PLUGINS = Constants.CORE_PLUGINS;
```

## Configuration Reloading

### Automatic Save

Server config is auto-saved periodically:

```java
// Every minute if changed
if (hytaleServerConfig.consumeHasChanged()) {
    HytaleServerConfig.save(hytaleServerConfig).join();
}
```

### Log Level Reloading

```java
HytaleLoggerBackend.reloadLogLevels();
```

## Example Configuration

### server-config.json

```json
{
    "serverName": "My Hytale Server",
    "maxPlayers": 100,

    "logLevels": {
        "com.hypixel.hytale": "INFO",
        "myplugin": "FINE"
    },

    "modules": {
        "weather": {
            "enabled": true,
            "defaultWeather": "clear"
        },
        "crafting": {
            "enabled": true,
            "instantCraft": false
        },
        "worldgen": {
            "enabled": true,
            "seed": 12345
        }
    },

    "rateLimits": {
        "packetsPerSecond": 1000,
        "bytesPerSecond": 1000000
    },

    "update": {
        "autoApply": "RESTART",
        "checkInterval": 3600
    }
}
```

### Plugin config.json

```json
{
    "enabled": true,
    "debugMode": false,
    "welcomeMessage": "Welcome to the server!",
    "maxItems": 64,
    "features": {
        "feature1": true,
        "feature2": false
    }
}
```

## Best Practices

1. **Use codecs** - Type-safe configuration
2. **Provide defaults** - Handle missing values
3. **Validate config** - Check constraints
4. **Document options** - Clear descriptions
5. **Hot-reload when possible** - Avoid restarts
6. **Backup configs** - Before modifications
7. **Use appropriate types** - Strong typing
