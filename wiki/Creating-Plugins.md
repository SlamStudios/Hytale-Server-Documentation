
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
import com.hypixel.hytale.server.core.plugin.PluginType;

public class MyPlugin extends JavaPlugin {

    public MyPlugin(JavaPluginInit init) {
        super(init);
    }

    @Override
    public PluginType getType() {
        return PluginType.PLUGIN;
    }

    @Override
    protected void setup() {
        getLogger().info("MyPlugin setup!");
        // Register commands, events, etc.
    }

    @Override
    protected void start() {
        getLogger().info("MyPlugin started!");
        // Start background tasks, etc.
    }

    @Override
    protected void shutdown() {
        getLogger().info("MyPlugin shutting down!");
        // Cleanup resources
    }
}
```

### 2. Create Plugin Manifest

Create `plugin.json` in your JAR root:

```json
{
    "name": "MyPlugin",
    "group": "com.example",
    "version": "1.0.0",
    "main": "com.example.myplugin.MyPlugin",
    "description": "My awesome plugin",
    "authors": ["YourName"],
    "dependencies": []
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
public class MyConfig {
    public static final BuilderCodec<MyConfig> CODEC = BuilderCodec.builder(MyConfig.class)
        .with("enabled", Codec.BOOLEAN, MyConfig::isEnabled, MyConfig.Builder::enabled)
        .with("message", Codec.STRING, MyConfig::getMessage, MyConfig.Builder::message)
        .with("maxValue", Codec.INT, MyConfig::getMaxValue, MyConfig.Builder::maxValue)
        .withDefault(MyConfig.Builder::new)
        .build();

    private final boolean enabled;
    private final String message;
    private final int maxValue;

    private MyConfig(boolean enabled, String message, int maxValue) {
        this.enabled = enabled;
        this.message = message;
        this.maxValue = maxValue;
    }

    public boolean isEnabled() { return enabled; }
    public String getMessage() { return message; }
    public int getMaxValue() { return maxValue; }

    public static class Builder {
        private boolean enabled = true;
        private String message = "Default message";
        private int maxValue = 100;

        public Builder enabled(boolean enabled) {
            this.enabled = enabled;
            return this;
        }

        public Builder message(String message) {
            this.message = message;
            return this;
        }

        public Builder maxValue(int maxValue) {
            this.maxValue = maxValue;
            return this;
        }

        public MyConfig build() {
            return new MyConfig(enabled, message, maxValue);
        }
    }
}
```

### Use Config in Plugin

```java
public class MyPlugin extends JavaPlugin {
    private Config<MyConfig> config;

    public MyPlugin(JavaPluginInit init) {
        super(init);
        // Must be before setup()
        config = withConfig("config", MyConfig.CODEC);
    }

    @Override
    protected void setup() {
        MyConfig cfg = config.get();
        if (!cfg.isEnabled()) {
            getLogger().info("Plugin disabled by config");
            return;
        }
        getLogger().info("Message: " + cfg.getMessage());
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

### Create Command

```java
public class MyCommand implements Command {
    private final MyPlugin plugin;

    public MyCommand(MyPlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public String getName() {
        return "mycommand";
    }

    @Override
    public String[] getAliases() {
        return new String[]{"mc"};
    }

    @Override
    public String getDescription() {
        return "My custom command";
    }

    @Override
    public String getUsage() {
        return "/mycommand <action> [args...]";
    }

    @Override
    public String getPermission() {
        return "myplugin.command";
    }

    @Override
    public void execute(CommandSender sender, String[] args) {
        if (args.length == 0) {
            sender.sendMessage("Usage: " + getUsage());
            return;
        }

        switch (args[0].toLowerCase()) {
            case "info":
                sender.sendMessage("MyPlugin v1.0.0");
                break;
            case "reload":
                if (sender.hasPermission("myplugin.reload")) {
                    plugin.reload();
                    sender.sendMessage("Config reloaded!");
                } else {
                    sender.sendMessage("No permission!");
                }
                break;
            default:
                sender.sendMessage("Unknown action: " + args[0]);
        }
    }

    @Override
    public List<String> tabComplete(CommandSender sender, String[] args) {
        if (args.length == 1) {
            return List.of("info", "reload").stream()
                .filter(s -> s.startsWith(args[0].toLowerCase()))
                .collect(Collectors.toList());
        }
        return Collections.emptyList();
    }
}
```

### Register Command

```java
@Override
protected void setup() {
    getCommandRegistry().register(new MyCommand(this));
}
```

## Events

### Listen to Events

```java
@Override
protected void setup() {
    EventRegistry events = getEventRegistry();

    // Simple listener
    events.register(PlayerJoinEvent.class, this::onPlayerJoin);

    // With priority
    events.register(EventPriority.HIGH, BlockBreakEvent.class, this::onBlockBreak);

    // Global listener (all keys)
    events.registerGlobal(SomeKeyedEvent.class, this::onAnyEvent);
}

private void onPlayerJoin(PlayerJoinEvent event) {
    Player player = event.getPlayer();
    player.sendMessage("Welcome, " + player.getUsername() + "!");
}

private void onBlockBreak(BlockBreakEvent event) {
    if (shouldPrevent(event)) {
        event.setCancelled(true);
    }
}
```

### Create Custom Events

```java
public class MyCustomEvent implements IEvent<Void>, ICancellable {
    private final Player player;
    private String customData;
    private boolean cancelled;

    public MyCustomEvent(Player player, String customData) {
        this.player = player;
        this.customData = customData;
    }

    public Player getPlayer() { return player; }
    public String getCustomData() { return customData; }
    public void setCustomData(String data) { this.customData = data; }

    @Override
    public boolean isCancelled() { return cancelled; }

    @Override
    public void setCancelled(boolean cancelled) { this.cancelled = cancelled; }
}
```

### Dispatch Events

```java
MyCustomEvent event = new MyCustomEvent(player, "data");
HytaleServer.get().getEventBus()
    .dispatchFor(MyCustomEvent.class)
    .dispatch(event);

if (!event.isCancelled()) {
    // Proceed with action
}
```

## Scheduled Tasks

```java
@Override
protected void setup() {
    TaskRegistry tasks = getTaskRegistry();

    // Repeating task
    tasks.scheduleRepeating(this::doPeriodicWork, 20, 20); // Every second (20 ticks)

    // Delayed task
    tasks.scheduleDelayed(this::doLaterWork, 100); // 5 seconds later
}

private void doPeriodicWork() {
    // Called every tick interval
}

private void doLaterWork() {
    // Called once after delay
}
```

## Entity Components

### Register Component

```java
public class MyComponent implements Component {
    private int value;

    public int getValue() { return value; }
    public void setValue(int value) { this.value = value; }
}

// In plugin
ComponentType<MyComponent> MY_COMPONENT;

@Override
protected void setup() {
    MY_COMPONENT = getEntityRegistry().registerComponent(
        MyComponent.class,
        MyComponent::new
    );
}
```

### Use Component

```java
// Add to entity
entity.addComponent(MY_COMPONENT, new MyComponent());

// Get from entity
MyComponent comp = entity.getComponent(MY_COMPONENT);
if (comp != null) {
    int value = comp.getValue();
}

// Remove from entity
entity.removeComponent(MY_COMPONENT);
```

## Block States

```java
public class MyBlockState implements TickableBlockState {
    public static final Codec<MyBlockState> CODEC = // ...

    private int counter;

    @Override
    public void onTick(World world, int x, int y, int z) {
        counter++;
        if (counter >= 100) {
            // Do something
            counter = 0;
        }
    }
}

// Register
@Override
protected void setup() {
    getBlockStateRegistry().register(MyBlockState.class, MyBlockState.CODEC);
}
```

## Assets

### Register Asset Type

```java
public class MyAsset extends JsonAsset<String> {
    public static final AssetCodec<MyAsset> CODEC = // ...

    private String property;

    // Getters, etc.
}

@Override
protected void setup() {
    getAssetRegistry().register("my_asset", MyAsset.class, MyAsset.CODEC);
}
```

## Logging

```java
private void someMethod() {
    HytaleLogger logger = getLogger();

    logger.info("Info message");
    logger.warning("Warning message");
    logger.severe("Error message");

    // With exception
    try {
        // ...
    } catch (Exception e) {
        logger.at(Level.SEVERE).withCause(e).log("Failed to do something");
    }

    // With formatting
    logger.info("Player %s joined from %s", playerName, address);
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
    "dependencies": ["RequiredPlugin", "AnotherPlugin"]
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
3. **Log appropriately** - Not too verbose, not too quiet
4. **Check state** - Before accessing other plugins
5. **Clean up** - In shutdown() method
6. **Use async** - For blocking operations
7. **Validate input** - Security first
8. **Document** - Commands, permissions, config

## Example: Complete Plugin

```java
package com.example.welcomeplugin;

import com.hypixel.hytale.server.core.plugin.*;
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.event.*;
import java.util.logging.Level;

public class WelcomePlugin extends JavaPlugin {
    private Config<WelcomeConfig> config;

    public WelcomePlugin(JavaPluginInit init) {
        super(init);
        config = withConfig("config", WelcomeConfig.CODEC);
    }

    @Override
    public PluginType getType() {
        return PluginType.PLUGIN;
    }

    @Override
    protected void setup() {
        // Register events
        getEventRegistry().register(PlayerJoinEvent.class, this::onJoin);
        getEventRegistry().register(PlayerQuitEvent.class, this::onQuit);

        // Register commands
        getCommandRegistry().register(new WelcomeCommand(this));

        getLogger().info("WelcomePlugin setup complete");
    }

    @Override
    protected void start() {
        getLogger().info("WelcomePlugin started with message: " +
            config.get().getWelcomeMessage());
    }

    @Override
    protected void shutdown() {
        getLogger().info("WelcomePlugin shutting down");
    }

    private void onJoin(PlayerJoinEvent event) {
        Player player = event.getPlayer();
        String message = config.get().getWelcomeMessage()
            .replace("{player}", player.getUsername());
        player.sendMessage(message);
    }

    private void onQuit(PlayerQuitEvent event) {
        if (config.get().isAnnounceQuit()) {
            // Announce to server
        }
    }

    public WelcomeConfig getConfig() {
        return config.get();
    }
}
```
