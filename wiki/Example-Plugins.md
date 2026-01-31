# Example Plugins

This page provides example plugins demonstrating Hytale server features. All code is verified against the decompiled server source.

## Table of Contents

- [Basic Plugin](#basic-plugin)
- [Event Listener Plugin](#event-listener-plugin)
- [Command Plugin](#command-plugin)
- [Configuration Plugin](#configuration-plugin)

---

## Basic Plugin

The simplest plugin structure.

### BasicPlugin.java

```java
package com.example.basic;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class BasicPlugin extends JavaPlugin {

    public BasicPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        getLogger().at(Level.INFO).log("BasicPlugin is setting up!");
    }

    @Override
    protected void start() {
        getLogger().at(Level.INFO).log("BasicPlugin has started!");
    }

    @Override
    protected void shutdown() {
        getLogger().at(Level.INFO).log("BasicPlugin is shutting down!");
    }
}
```

### plugin.json

```json
{
    "Group": "com.example",
    "Name": "BasicPlugin",
    "Version": "1.0.0",
    "Description": "A basic example plugin",
    "Main": "com.example.basic.BasicPlugin",
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

---

## Event Listener Plugin

Demonstrates listening to player events using verified event classes.

### EventPlugin.java

```java
package com.example.events;

import com.hypixel.hytale.event.EventPriority;
import com.hypixel.hytale.event.EventRegistry;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.event.events.BootEvent;
import com.hypixel.hytale.server.core.event.events.ShutdownEvent;
import com.hypixel.hytale.server.core.event.events.player.PlayerConnectEvent;
import com.hypixel.hytale.server.core.event.events.player.PlayerDisconnectEvent;
import com.hypixel.hytale.server.core.event.events.player.PlayerChatEvent;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class EventPlugin extends JavaPlugin {

    public EventPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        EventRegistry events = getEventRegistry();

        // Server lifecycle events
        events.register(BootEvent.class, this::onBoot);
        events.register(ShutdownEvent.class, this::onShutdown);

        // Player events
        events.register(PlayerConnectEvent.class, this::onPlayerConnect);
        events.register(PlayerDisconnectEvent.class, this::onPlayerDisconnect);
        
        // Chat event with HIGH priority (IAsyncEvent)
        events.registerAsync(EventPriority.HIGH, PlayerChatEvent.class, future -> {
            return future.thenApply(event -> {
                onPlayerChat(event);
                return event;
            });
        });

        getLogger().at(Level.INFO).log("Event listeners registered");
    }

    private void onBoot(BootEvent event) {
        getLogger().at(Level.INFO).log("Server has booted!");
    }

    private void onShutdown(ShutdownEvent event) {
        getLogger().at(Level.INFO).log("Server is shutting down!");
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
        // PlayerChatEvent is cancellable
        String content = event.getContent();
        PlayerRef sender = event.getSender();
        
        getLogger().at(Level.INFO).log("Chat from %s: %s", sender.getUsername(), content);
        
        // Example: filter content
        if (content.contains("badword")) {
            event.setCancelled(true);
        }
    }
}
```

---

## Command Plugin

Demonstrates creating commands using the actual AbstractCommand system.

### CommandPlugin.java

```java
package com.example.commands;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.command.system.CommandRegistry;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class CommandPlugin extends JavaPlugin {

    public CommandPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        CommandRegistry commands = getCommandRegistry();
        
        commands.registerCommand(new InfoCommand());
        commands.registerCommand(new GiveItemCommand());

        getLogger().at(Level.INFO).log("Commands registered");
    }
}
```

### InfoCommand.java

A simple command with no arguments:

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.basecommands.CommandBase;
import java.util.concurrent.CompletableFuture;
import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public class InfoCommand extends CommandBase {

    public InfoCommand() {
        super("info", "example.commands.info.description");
        addAliases("serverinfo", "si");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory() / 1024 / 1024;
        long usedMemory = (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024;

        context.sendMessage(Message.raw("=== Server Info ==="));
        context.sendMessage(Message.raw("Memory: " + usedMemory + "MB / " + maxMemory + "MB"));
        context.sendMessage(Message.raw("Java: " + System.getProperty("java.version")));
    }
}
```

### GiveItemCommand.java

A command with typed arguments:

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.DefaultArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import java.util.UUID;
import java.util.concurrent.CompletableFuture;
import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public class GiveItemCommand extends AbstractCommand {

    @Nonnull
    private final RequiredArg<UUID> playerArg;
    
    @Nonnull
    private final RequiredArg<String> itemArg;
    
    @Nonnull
    private final DefaultArg<Integer> amountArg;

    public GiveItemCommand() {
        super("giveitem", "example.commands.giveitem.description");
        
        // Define arguments
        this.playerArg = withRequiredArg("player", 
            "example.commands.giveitem.player.description", 
            ArgTypes.PLAYER_UUID);
            
        this.itemArg = withRequiredArg("item", 
            "example.commands.giveitem.item.description", 
            ArgTypes.STRING);
            
        this.amountArg = withDefaultArg("amount", 
            "example.commands.giveitem.amount.description", 
            ArgTypes.POSITIVE_INTEGER, 
            1,      // default value
            "1");   // default display
        
        // Require permission
        requirePermission("example.command.giveitem");
    }

    @Override
    @Nullable
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        UUID targetUuid = context.get(playerArg);
        String itemName = context.get(itemArg);
        int amount = context.get(amountArg);

        // Implementation would give item to player
        context.sendMessage(Message.raw("Giving " + amount + "x " + itemName + " to player"));
        
        return null; // Sync execution complete
    }
}
```

### SubcommandExample.java

A command with subcommands:

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.basecommands.CommandBase;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import javax.annotation.Nonnull;

public class AdminCommand extends CommandBase {

    public AdminCommand() {
        super("admin", "example.commands.admin.description");
        addAliases("adm");
        
        // Add subcommands
        addSubCommand(new ReloadSubcommand());
        addSubCommand(new StatusSubcommand());
        
        requirePermission("example.admin");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        // Called when no subcommand specified
        context.sendMessage(Message.raw("Usage: /admin <reload|status>"));
    }
}

class ReloadSubcommand extends CommandBase {
    
    public ReloadSubcommand() {
        super("reload", "example.commands.admin.reload.description");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        context.sendMessage(Message.raw("Reloading configuration..."));
        // Reload logic
        context.sendMessage(Message.raw("Done!"));
    }
}

class StatusSubcommand extends CommandBase {
    
    public StatusSubcommand() {
        super("status", "example.commands.admin.status.description");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        context.sendMessage(Message.raw("Server status: Running"));
    }
}
```

---

## Configuration Plugin

Demonstrates plugin configuration using codecs.

### ConfigPlugin.java

```java
package com.example.config;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.util.Config;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class ConfigPlugin extends JavaPlugin {

    private Config<PluginConfig> config;

    public ConfigPlugin(@Nonnull JavaPluginInit init) {
        super(init);
        // Config must be registered in constructor, before setup()
        this.config = withConfig("config", PluginConfig.CODEC);
    }

    @Override
    protected void setup() {
        PluginConfig cfg = config.get();
        
        if (!cfg.isEnabled()) {
            getLogger().at(Level.INFO).log("Plugin is disabled by configuration");
            return;
        }
        
        getLogger().at(Level.INFO).log("Loaded config - message: %s, maxValue: %d", 
            cfg.getMessage(), cfg.getMaxValue());
    }
    
    public PluginConfig getPluginConfig() {
        return config.get();
    }
}
```

### PluginConfig.java

```java
package com.example.config;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import javax.annotation.Nonnull;

public class PluginConfig {

    @Nonnull
    public static final BuilderCodec<PluginConfig> CODEC = BuilderCodec
        .builder(PluginConfig.class, Builder::new)
        .append(new KeyedCodec<>("enabled", Codec.BOOLEAN),
                (cfg, val) -> {}, PluginConfig::isEnabled)
        .add()
        .append(new KeyedCodec<>("message", Codec.STRING),
                (cfg, val) -> {}, PluginConfig::getMessage)
        .add()
        .append(new KeyedCodec<>("maxValue", Codec.INT),
                (cfg, val) -> {}, PluginConfig::getMaxValue)
        .add()
        .build();

    private final boolean enabled;
    private final String message;
    private final int maxValue;

    private PluginConfig(Builder builder) {
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

        public PluginConfig build() {
            return new PluginConfig(this);
        }
    }
}
```

Generated config file at `plugins/ConfigPlugin/config.json`:

```json
{
    "enabled": true,
    "message": "Default message",
    "maxValue": 100
}
```

---

## See Also

- [Plugin System](Plugin-System) - Plugin architecture details
- [Creating Plugins](Creating-Plugins) - Step-by-step plugin creation guide
- [Event System](Event-System) - Event handling documentation
- [Command System](Command-System) - Command system documentation
