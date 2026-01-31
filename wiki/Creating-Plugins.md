# Creating Plugins

This guide covers how to create custom plugins for the Hytale Server. Plugins allow you to extend server functionality with custom features.

## Prerequisites

- Java 25 JDK
- Understanding of [Plugin System](Plugin-System)
- Familiarity with [Event System](Event-System)

## Quick Start

### 1. Create Plugin Class

```java
package com.example.myplugin;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class MyPlugin extends JavaPlugin {

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        getLogger().at(Level.INFO).log("MyPlugin setup!");
        // Register commands, events, etc.
    }

    @Override
    protected void start() {
        getLogger().at(Level.INFO).log("MyPlugin started!");
        // Start background tasks, etc.
    }

    @Override
    protected void shutdown() {
        getLogger().at(Level.INFO).log("MyPlugin shutting down!");
        // Cleanup resources
    }
}
```

### 2. Create Plugin Manifest

Create `plugin.json` in your JAR root:

```json
{
    "Group": "com.example",
    "Name": "MyPlugin",
    "Version": "1.0.0",
    "Main": "com.example.myplugin.MyPlugin",
    "Description": "My awesome plugin",
    "Authors": [
        {
            "Name": "YourName"
        }
    ],
    "ServerVersion": ">=0.0.1",
    "DisabledByDefault": false,
    "IncludesAssetPack": false
}
```

### 3. Build and Deploy

1. Compile your plugin to a JAR
2. Place the JAR in the `plugins/` directory
3. Start the server

## Plugin Structure

### Recommended Project Layout

```
my-plugin/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/myplugin/
│       │       ├── MyPlugin.java
│       │       ├── commands/
│       │       ├── listeners/
│       │       ├── config/
│       │       └── util/
│       └── resources/
│           └── plugin.json
├── build.gradle
└── settings.gradle
```

## Configuration

### Define Config Class

```java
package com.example.myplugin;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import javax.annotation.Nonnull;

public class MyConfig {

    @Nonnull
    public static final BuilderCodec<MyConfig> CODEC = BuilderCodec
        .builder(MyConfig.class, Builder::new)
        .append(new KeyedCodec<>("enabled", Codec.BOOLEAN),
                (cfg, val) -> {}, MyConfig::isEnabled)
        .add()
        .append(new KeyedCodec<>("message", Codec.STRING),
                (cfg, val) -> {}, MyConfig::getMessage)
        .add()
        .append(new KeyedCodec<>("maxValue", Codec.INT),
                (cfg, val) -> {}, MyConfig::getMaxValue)
        .add()
        .build();

    private final boolean enabled;
    private final String message;
    private final int maxValue;

    private MyConfig(Builder builder) {
        this.enabled = builder.enabled;
        this.message = builder.message;
        this.maxValue = builder.maxValue;
    }

    public boolean isEnabled() { return enabled; }
    public String getMessage() { return message; }
    public int getMaxValue() { return maxValue; }

    public static class Builder {
        private boolean enabled = true;
        private String message = "Default message";
        private int maxValue = 100;

        public MyConfig build() {
            return new MyConfig(this);
        }
    }
}
```

### Use Config in Plugin

```java
public class MyPlugin extends JavaPlugin {
    private Config<MyConfig> config;

    public MyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
        // Must be called in constructor, before setup()
        config = withConfig("config", MyConfig.CODEC);
    }

    @Override
    protected void setup() {
        MyConfig cfg = config.get();
        if (!cfg.isEnabled()) {
            getLogger().at(Level.INFO).log("Plugin disabled by config");
            return;
        }
        getLogger().at(Level.INFO).log("Message: %s", cfg.getMessage());
    }
}
```

### config.json

Created automatically in `plugins/MyPlugin/config.json`:

```json
{
    "enabled": true,
    "message": "Default message",
    "maxValue": 100
}
```

## Commands

See [Command System](Command-System) for full documentation.

### Simple Command (CommandBase)

Use `CommandBase` for synchronous commands:

```java
package com.example.myplugin.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.basecommands.CommandBase;
import javax.annotation.Nonnull;

public class InfoCommand extends CommandBase {

    public InfoCommand() {
        super("info", "myplugin.commands.info.description");
        addAliases("serverinfo");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        Runtime runtime = Runtime.getRuntime();
        long usedMB = (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024;
        long maxMB = runtime.maxMemory() / 1024 / 1024;
        
        context.sendMessage(Message.raw("Memory: " + usedMB + "MB / " + maxMB + "MB"));
        context.sendMessage(Message.raw("Java: " + System.getProperty("java.version")));
    }
}
```

### Command with Arguments (AbstractCommand)

Use `AbstractCommand` for commands with typed arguments:

```java
package com.example.myplugin.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.DefaultArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import java.util.UUID;
import java.util.concurrent.CompletableFuture;
import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public class GiveCommand extends AbstractCommand {

    @Nonnull
    private final RequiredArg<UUID> playerArg;
    
    @Nonnull
    private final RequiredArg<String> itemArg;
    
    @Nonnull
    private final DefaultArg<Integer> amountArg;

    public GiveCommand() {
        super("give", "myplugin.commands.give.description");
        
        this.playerArg = withRequiredArg("player", 
            "myplugin.commands.give.player.description", 
            ArgTypes.PLAYER_UUID);
            
        this.itemArg = withRequiredArg("item", 
            "myplugin.commands.give.item.description", 
            ArgTypes.STRING);
            
        this.amountArg = withDefaultArg("amount", 
            "myplugin.commands.give.amount.description", 
            ArgTypes.POSITIVE_INTEGER, 
            1,      // default value
            "1");   // default display
        
        requirePermission("myplugin.command.give");
    }

    @Override
    @Nullable
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        UUID targetUuid = context.get(playerArg);
        String itemName = context.get(itemArg);
        int amount = context.get(amountArg);

        // Implementation here
        context.sendMessage(Message.raw("Giving " + amount + "x " + itemName));
        
        return null;
    }
}
```

### Register Commands

```java
@Override
protected void setup() {
    getCommandRegistry().registerCommand(new InfoCommand());
    getCommandRegistry().registerCommand(new GiveCommand());
}
```

## Events

See [Event System](Event-System) and [Event Types](Event-Types) for full documentation.

### Listen to Events

```java
@Override
protected void setup() {
    EventRegistry events = getEventRegistry();

    // Server lifecycle events
    events.register(BootEvent.class, this::onBoot);
    events.register(ShutdownEvent.class, this::onShutdown);

    // Player events (synchronous)
    events.register(PlayerConnectEvent.class, this::onPlayerConnect);
    events.register(PlayerDisconnectEvent.class, this::onPlayerDisconnect);
    
    // Async events require registerAsync
    events.registerAsync(PlayerChatEvent.class, future -> {
        return future.thenApply(event -> {
            onPlayerChat(event);
            return event;
        });
    });
}

private void onBoot(BootEvent event) {
    getLogger().at(Level.INFO).log("Server booted!");
}

private void onShutdown(ShutdownEvent event) {
    getLogger().at(Level.INFO).log("Server shutting down!");
}

private void onPlayerConnect(PlayerConnectEvent event) {
    PlayerRef playerRef = event.getPlayerRef();
    getLogger().at(Level.INFO).log("Player connecting: %s", playerRef.getUsername());
}

private void onPlayerDisconnect(PlayerDisconnectEvent event) {
    PlayerRef playerRef = event.getPlayerRef();
    getLogger().at(Level.INFO).log("Player disconnected: %s", playerRef.getUsername());
}

private void onPlayerChat(PlayerChatEvent event) {
    // PlayerChatEvent implements ICancellable
    if (event.getContent().contains("badword")) {
        event.setCancelled(true);
    }
}
```

### Create Custom Events

```java
package com.example.myplugin.events;

import com.hypixel.hytale.event.IEvent;
import com.hypixel.hytale.event.ICancellable;
import javax.annotation.Nonnull;

public class MyCustomEvent implements IEvent<Void>, ICancellable {
    private final String data;
    private boolean cancelled;

    public MyCustomEvent(@Nonnull String data) {
        this.data = data;
    }

    @Nonnull
    public String getData() { return data; }

    @Override
    public boolean isCancelled() { return cancelled; }

    @Override
    public void setCancelled(boolean cancelled) { this.cancelled = cancelled; }
}
```

### Dispatch Events

```java
import com.hypixel.hytale.server.core.HytaleServer;

MyCustomEvent event = new MyCustomEvent("some data");
HytaleServer.get().getEventBus()
    .dispatchFor(MyCustomEvent.class)
    .dispatch(event);

if (!event.isCancelled()) {
    // Proceed with action
}
```

## Logging

```java
import com.hypixel.hytale.logger.HytaleLogger;
import java.util.logging.Level;

private void someMethod() {
    HytaleLogger logger = getLogger();

    // Simple logging
    logger.at(Level.INFO).log("Info message");
    logger.at(Level.WARNING).log("Warning message");
    logger.at(Level.SEVERE).log("Error message");

    // With formatting
    logger.at(Level.INFO).log("Player %s connected from %s", playerName, address);

    // With exception
    try {
        // ...
    } catch (Exception e) {
        logger.at(Level.SEVERE).withCause(e).log("Failed to do something");
    }
}
```

## Error Handling

```java
@Override
protected void setup() {
    try {
        initializeFeature();
    } catch (Exception e) {
        getLogger().at(Level.SEVERE).withCause(e).log("Failed to initialize feature");
        // Feature gracefully degrades
    }
}
```

## Dependencies

### Declaring Dependencies

In `plugin.json`:

```json
{
    "Dependencies": ["RequiredPlugin", "AnotherPlugin"]
}
```

### Optional Dependencies

```java
@Override
protected void start() {
    PluginManager pm = HytaleServer.get().getPluginManager();

    PluginBase otherPlugin = pm.getPlugin("OptionalPlugin");
    if (otherPlugin != null && otherPlugin.isEnabled()) {
        // Integrate with optional plugin
    }
}
```

## Best Practices

1. **Use registries** - Automatic cleanup on disable
2. **Handle errors** - Don't crash the server
3. **Log appropriately** - Use Level.INFO for normal messages, Level.FINE for debug
4. **Check state** - Before accessing other plugins
5. **Clean up** - In shutdown() method
6. **Validate input** - Security first
7. **Document** - Commands, permissions, config

## See Also

- [Plugin System](Plugin-System) - Plugin architecture details
- [Event System](Event-System) - Event handling documentation
- [Event Types](Event-Types) - Available event types
- [Command System](Command-System) - Command system documentation
- [Example Plugins](Example-Plugins) - Complete working examples
