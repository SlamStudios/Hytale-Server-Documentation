
This page provides complete, working example plugins demonstrating various Hytale server features. Use these as templates for your own plugins.

## Table of Contents

- [Basic Plugin](#basic-plugin)
- [Welcome Plugin](#welcome-plugin)
- [Custom Command Plugin](#custom-command-plugin)
- [Event Listener Plugin](#event-listener-plugin)
- [Custom Entity Component Plugin](#custom-entity-component-plugin)
- [World Manipulation Plugin](#world-manipulation-plugin)
- [Scheduled Tasks Plugin](#scheduled-tasks-plugin)
- [Economy Plugin](#economy-plugin)
- [Teleportation Plugin](#teleportation-plugin)

---

## Basic Plugin

The simplest possible plugin structure.

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
## Manifest File

### manifest.json

```json
{
  "Group": "com.example",
  "Name": "BasicPlugin",
  "Version": "1.0.0",
  "Description": "A basic example plugin",
  "Main": "com.example.basic.BasicPlugin",
  "Authors": [
    {
      "Name": "YourName",
      "Email": "you@example.com",
      "Url": "https://example.com"
    }
  ],
  "Website": "https://example.com/basicplugin",
  "ServerVersion": ">=0.0.1",
  "DisabledByDefault": false,
  "IncludesAssetPack": false
}
```

---

## Welcome Plugin

Welcomes players when they join and announces when they leave.

### WelcomePlugin.java

```java
package com.example.welcome;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.BuilderCodec;
import com.hypixel.hytale.event.EventRegistry;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.plugin.Config;
import com.hypixel.hytale.server.core.universe.Universe;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.server.core.universe.events.PlayerJoinEvent;
import com.hypixel.hytale.server.core.universe.events.PlayerQuitEvent;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class WelcomePlugin extends JavaPlugin {

    private Config<WelcomeConfig> config;

    public WelcomePlugin(@Nonnull JavaPluginInit init) {
        super(init);
        config = withConfig("config", WelcomeConfig.CODEC);
    }

    @Override
    protected void setup() {
        EventRegistry eventRegistry = getEventRegistry();

        // Register join event
        eventRegistry.register(PlayerJoinEvent.class, this::onPlayerJoin);

        // Register quit event
        eventRegistry.register(PlayerQuitEvent.class, this::onPlayerQuit);

        getLogger().at(Level.INFO).log("WelcomePlugin setup complete!");
    }

    @Override
    protected void start() {
        WelcomeConfig cfg = config.get();
        getLogger().at(Level.INFO).log("Welcome message: %s", cfg.getJoinMessage());
    }

    private void onPlayerJoin(PlayerJoinEvent event) {
        PlayerRef playerRef = event.getPlayerRef();
        WelcomeConfig cfg = config.get();

        // Send welcome message to joining player
        String welcomeMsg = cfg.getJoinMessage()
            .replace("{player}", playerRef.getUsername())
            .replace("{online}", String.valueOf(Universe.get().getPlayerCount()));
        playerRef.sendMessage(Message.raw(welcomeMsg));

        // Broadcast join to all players if enabled
        if (cfg.isBroadcastJoin()) {
            String broadcastMsg = cfg.getJoinBroadcast()
                .replace("{player}", playerRef.getUsername());
            broadcastToAll(broadcastMsg);
        }
    }

    private void onPlayerQuit(PlayerQuitEvent event) {
        PlayerRef playerRef = event.getPlayerRef();
        WelcomeConfig cfg = config.get();

        if (cfg.isBroadcastQuit()) {
            String quitMsg = cfg.getQuitBroadcast()
                .replace("{player}", playerRef.getUsername());
            broadcastToAll(quitMsg);
        }
    }

    private void broadcastToAll(String message) {
        Universe universe = Universe.get();
        for (PlayerRef player : universe.getPlayers()) {
            player.sendMessage(Message.raw(message));
        }
    }

    @Override
    protected void shutdown() {
        getLogger().at(Level.INFO).log("WelcomePlugin shutting down");
    }
}
```

### WelcomeConfig.java

```java
package com.example.welcome;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.BuilderCodec;
import javax.annotation.Nonnull;

public class WelcomeConfig {

    @Nonnull
    public static final BuilderCodec<WelcomeConfig> CODEC = BuilderCodec
        .builder(WelcomeConfig.class, Builder::new)
        .append(new KeyedCodec<>("joinMessage", Codec.STRING),
                (cfg, val) -> {}, WelcomeConfig::getJoinMessage)
        .add()
        .append(new KeyedCodec<>("joinBroadcast", Codec.STRING),
                (cfg, val) -> {}, WelcomeConfig::getJoinBroadcast)
        .add()
        .append(new KeyedCodec<>("quitBroadcast", Codec.STRING),
                (cfg, val) -> {}, WelcomeConfig::getQuitBroadcast)
        .add()
        .append(new KeyedCodec<>("broadcastJoin", Codec.BOOLEAN),
                (cfg, val) -> {}, WelcomeConfig::isBroadcastJoin)
        .add()
        .append(new KeyedCodec<>("broadcastQuit", Codec.BOOLEAN),
                (cfg, val) -> {}, WelcomeConfig::isBroadcastQuit)
        .add()
        .build();

    private final String joinMessage;
    private final String joinBroadcast;
    private final String quitBroadcast;
    private final boolean broadcastJoin;
    private final boolean broadcastQuit;

    private WelcomeConfig(Builder builder) {
        this.joinMessage = builder.joinMessage;
        this.joinBroadcast = builder.joinBroadcast;
        this.quitBroadcast = builder.quitBroadcast;
        this.broadcastJoin = builder.broadcastJoin;
        this.broadcastQuit = builder.broadcastQuit;
    }

    public String getJoinMessage() { return joinMessage; }
    public String getJoinBroadcast() { return joinBroadcast; }
    public String getQuitBroadcast() { return quitBroadcast; }
    public boolean isBroadcastJoin() { return broadcastJoin; }
    public boolean isBroadcastQuit() { return broadcastQuit; }

    public static class Builder {
        private String joinMessage = "Welcome to the server, {player}! There are {online} players online.";
        private String joinBroadcast = "{player} has joined the server!";
        private String quitBroadcast = "{player} has left the server.";
        private boolean broadcastJoin = true;
        private boolean broadcastQuit = true;

        public WelcomeConfig build() {
            return new WelcomeConfig(this);
        }
    }
}
```

---

## Custom Command Plugin

Demonstrates creating commands with subcommands and arguments.

### CommandPlugin.java

```java
package com.example.commands;

import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandRegistry;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class CommandPlugin extends JavaPlugin {

    public CommandPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        CommandRegistry commandRegistry = getCommandRegistry();

        // Register commands
        commandRegistry.registerCommand(new AdminCommand());
        commandRegistry.registerCommand(new HealCommand());
        commandRegistry.registerCommand(new FeedCommand());

        getLogger().at(Level.INFO).log("Commands registered!");
    }
}
```

### AdminCommand.java

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.basecommands.CommandBase;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import javax.annotation.Nonnull;

public class AdminCommand extends CommandBase {

    public AdminCommand() {
        super("admin", "server.commands.admin.desc");

        // Add subcommands
        addSubCommand(new InfoSubcommand());
        addSubCommand(new ReloadSubcommand());
        addSubCommand(new BroadcastSubcommand());

        // Add aliases
        addAliases("adm", "a");

        // Require permission
        requirePermission("commandplugin.admin");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        // Show help when no subcommand specified
        context.sendMessage(Message.raw("Available subcommands: info, reload, broadcast"));
    }
}

class InfoSubcommand extends CommandBase {

    public InfoSubcommand() {
        super("info", "server.commands.admin.info.desc");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory() / 1024 / 1024;
        long usedMemory = (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024;

        context.sendMessage(Message.raw("=== Server Info ==="));
        context.sendMessage(Message.raw("Memory: " + usedMemory + "MB / " + maxMemory + "MB"));
        context.sendMessage(Message.raw("Java: " + System.getProperty("java.version")));
        context.sendMessage(Message.raw("OS: " + System.getProperty("os.name")));
    }
}

class ReloadSubcommand extends CommandBase {

    public ReloadSubcommand() {
        super("reload", "server.commands.admin.reload.desc");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        context.sendMessage(Message.raw("Reloading configuration..."));
        // Reload logic here
        context.sendMessage(Message.raw("Configuration reloaded!"));
    }
}

class BroadcastSubcommand extends CommandBase {

    @Nonnull
    private final RequiredArg<String> messageArg;

    public BroadcastSubcommand() {
        super("broadcast", "server.commands.admin.broadcast.desc");
        messageArg = withRequiredArg("message", "server.commands.admin.broadcast.message.desc", ArgTypes.STRING);
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        String message = messageArg.get(context);
        // Broadcast to all players
        context.sendMessage(Message.raw("Broadcasted: " + message));
    }
}
```

### HealCommand.java

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.basecommands.AbstractPlayerCommand;
import com.hypixel.hytale.server.core.command.system.arguments.system.DefaultArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.UUID;
import javax.annotation.Nonnull;

public class HealCommand extends AbstractPlayerCommand {

    @Nonnull
    private final DefaultArg<UUID> targetArg;

    public HealCommand() {
        super("heal", "server.commands.heal.desc");
        addAliases("h");
        targetArg = withDefaultArg("player", "server.commands.heal.player.desc",
                                   ArgTypes.PLAYER_UUID, null, "self");
        requirePermission("commandplugin.heal");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        UUID targetUuid = targetArg.get(context);

        if (targetUuid == null) {
            // Heal self
            context.sendMessage(Message.raw("You have been healed!"));
        } else {
            // Heal target
            context.sendMessage(Message.raw("Player has been healed!"));
        }
    }
}
```

### FeedCommand.java

```java
package com.example.commands;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.basecommands.AbstractPlayerCommand;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import javax.annotation.Nonnull;

public class FeedCommand extends AbstractPlayerCommand {

    public FeedCommand() {
        super("feed", "server.commands.feed.desc");
        requirePermission("commandplugin.feed");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        context.sendMessage(Message.raw("Your hunger has been restored!"));
    }
}
```

---

## Event Listener Plugin

Demonstrates listening to various game events.

### EventListenerPlugin.java

```java
package com.example.events;

import com.hypixel.hytale.event.EventPriority;
import com.hypixel.hytale.event.EventRegistry;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.server.core.universe.events.PlayerJoinEvent;
import com.hypixel.hytale.server.core.universe.world.events.BlockBreakEvent;
import com.hypixel.hytale.server.core.universe.world.events.BlockPlaceEvent;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class EventListenerPlugin extends JavaPlugin {

    // Track protected players
    private final Set<UUID> protectedPlayers = new HashSet<>();

    public EventListenerPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        EventRegistry eventRegistry = getEventRegistry();

        // Block break event - prevent breaking certain blocks
        eventRegistry.register(
            EventPriority.HIGH,
            BlockBreakEvent.class,
            this::onBlockBreak
        );

        // Block place event - log placements (MONITOR priority = observe only)
        eventRegistry.register(
            EventPriority.MONITOR,
            BlockPlaceEvent.class,
            this::onBlockPlace
        );

        // Player join event
        eventRegistry.register(
            EventPriority.NORMAL,
            PlayerJoinEvent.class,
            this::onPlayerJoin
        );

        getLogger().at(Level.INFO).log("Event listeners registered!");
    }

    private void onBlockBreak(BlockBreakEvent event) {
        // Example: Prevent breaking bedrock
        if (event.getBlockType().getName().equals("bedrock")) {
            event.setCancelled(true);
            PlayerRef player = event.getPlayerRef();
            if (player != null) {
                player.sendMessage(Message.raw("You cannot break bedrock!"));
            }
        }
    }

    private void onBlockPlace(BlockPlaceEvent event) {
        // Monitor only - just log, don't modify
        PlayerRef player = event.getPlayerRef();
        if (player != null) {
            getLogger().at(Level.FINE).log(
                "Player %s placed %s at (%d, %d, %d)",
                player.getUsername(),
                event.getBlockType().getName(),
                event.getX(), event.getY(), event.getZ()
            );
        }
    }

    private void onPlayerJoin(PlayerJoinEvent event) {
        PlayerRef player = event.getPlayerRef();
        getLogger().at(Level.INFO).log("Player joined: %s", player.getUsername());
    }

    public void toggleProtection(UUID uuid) {
        if (protectedPlayers.contains(uuid)) {
            protectedPlayers.remove(uuid);
        } else {
            protectedPlayers.add(uuid);
        }
    }

    public boolean isProtected(UUID uuid) {
        return protectedPlayers.contains(uuid);
    }
}
```

---

## Custom Entity Component Plugin

Demonstrates creating and using custom ECS components.

### ComponentPlugin.java

```java
package com.example.components;

import com.hypixel.hytale.component.Component;
import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.universe.world.storage.EntityStore;
import java.util.logging.Level;
import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public class ComponentPlugin extends JavaPlugin {

    // Component types
    private ComponentType<EntityStore, ManaComponent> manaComponentType;
    private ComponentType<EntityStore, PoisonComponent> poisonComponentType;
    private ComponentType<EntityStore, ExperienceComponent> experienceComponentType;

    public ComponentPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        // Register components with the EntityStore registry
        manaComponentType = EntityStore.REGISTRY.registerComponent(
            ManaComponent.class,
            ManaComponent::new
        );

        poisonComponentType = EntityStore.REGISTRY.registerComponent(
            PoisonComponent.class,
            PoisonComponent::new
        );

        experienceComponentType = EntityStore.REGISTRY.registerComponent(
            ExperienceComponent.class,
            ExperienceComponent::new
        );

        getLogger().at(Level.INFO).log("Custom components registered!");
    }

    public ComponentType<EntityStore, ManaComponent> getManaComponentType() {
        return manaComponentType;
    }

    public ComponentType<EntityStore, PoisonComponent> getPoisonComponentType() {
        return poisonComponentType;
    }

    public ComponentType<EntityStore, ExperienceComponent> getExperienceComponentType() {
        return experienceComponentType;
    }
}

// Mana component - stores magical energy
class ManaComponent implements Component<EntityStore> {
    private float mana;
    private float maxMana;
    private float regenRate;

    public ManaComponent() {
        this.mana = 100f;
        this.maxMana = 100f;
        this.regenRate = 1f;
    }

    public float getMana() { return mana; }
    public void setMana(float mana) { this.mana = Math.min(mana, maxMana); }
    public float getMaxMana() { return maxMana; }
    public void setMaxMana(float maxMana) { this.maxMana = maxMana; }
    public float getRegenRate() { return regenRate; }
    public void setRegenRate(float rate) { this.regenRate = rate; }

    public boolean useMana(float amount) {
        if (mana >= amount) {
            mana -= amount;
            return true;
        }
        return false;
    }

    public void regenerate(float deltaTime) {
        mana = Math.min(mana + regenRate * deltaTime, maxMana);
    }

    @Nullable
    @Override
    public Component<EntityStore> clone() {
        ManaComponent clone = new ManaComponent();
        clone.mana = this.mana;
        clone.maxMana = this.maxMana;
        clone.regenRate = this.regenRate;
        return clone;
    }
}

// Poison component - damage over time
class PoisonComponent implements Component<EntityStore> {
    private float damagePerSecond;
    private float remainingDuration;

    public PoisonComponent() {
        this.damagePerSecond = 0f;
        this.remainingDuration = 0f;
    }

    public void applyPoison(float damage, float duration) {
        this.damagePerSecond = damage;
        this.remainingDuration = duration;
    }

    public float getDamagePerSecond() { return damagePerSecond; }
    public float getRemainingDuration() { return remainingDuration; }

    public void tick(float deltaTime) {
        remainingDuration -= deltaTime;
        if (remainingDuration <= 0) {
            damagePerSecond = 0;
            remainingDuration = 0;
        }
    }

    public boolean isPoisoned() {
        return remainingDuration > 0;
    }

    @Nullable
    @Override
    public Component<EntityStore> clone() {
        PoisonComponent clone = new PoisonComponent();
        clone.damagePerSecond = this.damagePerSecond;
        clone.remainingDuration = this.remainingDuration;
        return clone;
    }
}

// Experience component - leveling system
class ExperienceComponent implements Component<EntityStore> {
    private int level;
    private long experience;
    private long experienceToNextLevel;

    public ExperienceComponent() {
        this.level = 1;
        this.experience = 0;
        this.experienceToNextLevel = 100;
    }

    public int getLevel() { return level; }
    public long getExperience() { return experience; }
    public long getExperienceToNextLevel() { return experienceToNextLevel; }

    public boolean addExperience(long amount) {
        experience += amount;
        if (experience >= experienceToNextLevel) {
            levelUp();
            return true;
        }
        return false;
    }

    private void levelUp() {
        level++;
        experience -= experienceToNextLevel;
        experienceToNextLevel = (long) (experienceToNextLevel * 1.5);
    }

    @Nullable
    @Override
    public Component<EntityStore> clone() {
        ExperienceComponent clone = new ExperienceComponent();
        clone.level = this.level;
        clone.experience = this.experience;
        clone.experienceToNextLevel = this.experienceToNextLevel;
        return clone;
    }
}
```

---

## World Manipulation Plugin

Demonstrates block and world manipulation.

### WorldPlugin.java

```java
package com.example.world;

import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.command.system.AbstractCommand;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.CommandRegistry;
import com.hypixel.hytale.server.core.command.system.basecommands.AbstractPlayerCommand;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.asset.type.blocktype.config.BlockType;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.server.core.universe.Universe;
import com.hypixel.hytale.server.core.universe.world.World;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class WorldPlugin extends JavaPlugin {

    // Store clipboard data per player
    private final Map<UUID, ClipboardData> clipboards = new ConcurrentHashMap<>();

    public WorldPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        CommandRegistry commandRegistry = getCommandRegistry();

        commandRegistry.registerCommand(new FillCommand());
        commandRegistry.registerCommand(new ReplaceCommand());

        getLogger().at(Level.INFO).log("World manipulation commands registered!");
    }

    public void setClipboard(UUID player, ClipboardData data) {
        clipboards.put(player, data);
    }

    public ClipboardData getClipboard(UUID player) {
        return clipboards.get(player);
    }
}

// Clipboard data structure
class ClipboardData {
    private final BlockType[][][] blocks;
    private final int sizeX, sizeY, sizeZ;

    public ClipboardData(BlockType[][][] blocks, int sizeX, int sizeY, int sizeZ) {
        this.blocks = blocks;
        this.sizeX = sizeX;
        this.sizeY = sizeY;
        this.sizeZ = sizeZ;
    }

    public BlockType getBlock(int x, int y, int z) {
        return blocks[x][y][z];
    }

    public int getSizeX() { return sizeX; }
    public int getSizeY() { return sizeY; }
    public int getSizeZ() { return sizeZ; }
}

// Fill command - fill a region with blocks
class FillCommand extends AbstractPlayerCommand {

    @Nonnull private final RequiredArg<Integer> x1Arg;
    @Nonnull private final RequiredArg<Integer> y1Arg;
    @Nonnull private final RequiredArg<Integer> z1Arg;
    @Nonnull private final RequiredArg<Integer> x2Arg;
    @Nonnull private final RequiredArg<Integer> y2Arg;
    @Nonnull private final RequiredArg<Integer> z2Arg;
    @Nonnull private final RequiredArg<String> blockTypeArg;

    public FillCommand() {
        super("fill", "server.commands.fill.desc");
        x1Arg = withRequiredArg("x1", "server.commands.fill.x1.desc", ArgTypes.INTEGER);
        y1Arg = withRequiredArg("y1", "server.commands.fill.y1.desc", ArgTypes.INTEGER);
        z1Arg = withRequiredArg("z1", "server.commands.fill.z1.desc", ArgTypes.INTEGER);
        x2Arg = withRequiredArg("x2", "server.commands.fill.x2.desc", ArgTypes.INTEGER);
        y2Arg = withRequiredArg("y2", "server.commands.fill.y2.desc", ArgTypes.INTEGER);
        z2Arg = withRequiredArg("z2", "server.commands.fill.z2.desc", ArgTypes.INTEGER);
        blockTypeArg = withRequiredArg("blockType", "server.commands.fill.block.desc", ArgTypes.STRING);
        requirePermission("worldplugin.fill");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        int x1 = x1Arg.get(context);
        int y1 = y1Arg.get(context);
        int z1 = z1Arg.get(context);
        int x2 = x2Arg.get(context);
        int y2 = y2Arg.get(context);
        int z2 = z2Arg.get(context);
        String blockTypeName = blockTypeArg.get(context);

        // Get world from sender
        World world = sender.getWorld();
        if (world == null) {
            context.sendMessage(Message.raw("Could not determine world!"));
            return;
        }

        // Normalize coordinates
        int minX = Math.min(x1, x2), maxX = Math.max(x1, x2);
        int minY = Math.min(y1, y2), maxY = Math.max(y1, y2);
        int minZ = Math.min(z1, z2), maxZ = Math.max(z1, z2);

        int count = (maxX - minX + 1) * (maxY - minY + 1) * (maxZ - minZ + 1);

        // Execute block fill on the world thread
        world.execute(() -> {
            // Fill implementation would go here
        });

        context.sendMessage(Message.raw("Filling " + count + " blocks with " + blockTypeName));
    }
}

// Replace command - replace blocks in a region
class ReplaceCommand extends AbstractPlayerCommand {

    public ReplaceCommand() {
        super("replace", "server.commands.replace.desc");
        requirePermission("worldplugin.replace");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        context.sendMessage(Message.raw("Replace command executed!"));
    }
}
```

---

## Scheduled Tasks Plugin

Demonstrates periodic and delayed task scheduling.

### TasksPlugin.java

```java
package com.example.tasks;

import com.hypixel.hytale.server.core.HytaleServer;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.task.TaskRegistry;
import com.hypixel.hytale.server.core.universe.Universe;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.*;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ScheduledFuture;
import java.util.concurrent.TimeUnit;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class TasksPlugin extends JavaPlugin {

    private ScheduledFuture<?> autoSaveTask;
    private ScheduledFuture<?> announcementTask;

    private final List<String> announcements = Arrays.asList(
        "Welcome to our server!",
        "Join our Discord for updates!",
        "Remember to vote for the server!",
        "Check out /help for commands!"
    );
    private int announcementIndex = 0;

    public TasksPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Override
    protected void setup() {
        getLogger().at(Level.INFO).log("Setting up scheduled tasks...");
    }

    @Override
    protected void start() {
        TaskRegistry taskRegistry = getTaskRegistry();

        // Auto-save every 5 minutes (300000 milliseconds)
        autoSaveTask = HytaleServer.SCHEDULED_EXECUTOR.scheduleWithFixedDelay(
            this::autoSave,
            300000,  // Initial delay (5 min)
            300000,  // Period (5 min)
            TimeUnit.MILLISECONDS
        );
        taskRegistry.registerTask((ScheduledFuture<Void>) autoSaveTask);

        // Announcements every 3 minutes
        announcementTask = HytaleServer.SCHEDULED_EXECUTOR.scheduleWithFixedDelay(
            this::broadcastAnnouncement,
            180000,  // 3 min
            180000,
            TimeUnit.MILLISECONDS
        );
        taskRegistry.registerTask((ScheduledFuture<Void>) announcementTask);

        // One-time delayed task - welcome message 10 seconds after start
        CompletableFuture<Void> startupTask = CompletableFuture.runAsync(() -> {
            try {
                Thread.sleep(10000);
                getLogger().at(Level.INFO).log("Server is now fully ready!");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        taskRegistry.registerTask(startupTask);

        getLogger().at(Level.INFO).log("Scheduled tasks started!");
    }

    private void autoSave() {
        getLogger().at(Level.INFO).log("Running auto-save...");

        Universe universe = Universe.get();
        if (universe != null) {
            universe.getWorlds().values().forEach(world -> {
                getLogger().at(Level.FINE).log("Saving world: %s", world.getName());
            });
        }

        getLogger().at(Level.INFO).log("Auto-save complete!");
    }

    private void broadcastAnnouncement() {
        String announcement = announcements.get(announcementIndex);
        announcementIndex = (announcementIndex + 1) % announcements.size();

        Universe universe = Universe.get();
        if (universe != null) {
            for (PlayerRef player : universe.getPlayers()) {
                player.sendMessage(Message.raw("[Announcement] " + announcement));
            }
        }
    }

    @Override
    protected void shutdown() {
        // Cancel scheduled tasks
        if (autoSaveTask != null) {
            autoSaveTask.cancel(false);
        }
        if (announcementTask != null) {
            announcementTask.cancel(false);
        }

        getLogger().at(Level.INFO).log("Scheduled tasks stopped!");
    }
}
```

---

## Economy Plugin

A complete economy system with balance management.

### EconomyPlugin.java

```java
package com.example.economy;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.BuilderCodec;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.plugin.Config;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.CommandRegistry;
import com.hypixel.hytale.server.core.command.system.basecommands.AbstractPlayerCommand;
import com.hypixel.hytale.server.core.command.system.basecommands.CommandBase;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.logging.Level;
import javax.annotation.Nonnull;

public class EconomyPlugin extends JavaPlugin {

    private static EconomyPlugin instance;
    private Config<EconomyConfig> config;
    private final Map<UUID, Double> balances = new ConcurrentHashMap<>();

    public EconomyPlugin(@Nonnull JavaPluginInit init) {
        super(init);
        config = withConfig("config", EconomyConfig.CODEC);
    }

    @Nonnull
    public static EconomyPlugin get() {
        return instance;
    }

    @Override
    protected void setup() {
        instance = this;

        CommandRegistry commandRegistry = getCommandRegistry();

        // Register commands
        commandRegistry.registerCommand(new BalanceCommand());
        commandRegistry.registerCommand(new PayCommand());
        commandRegistry.registerCommand(new EcoCommand());

        getLogger().at(Level.INFO).log("Economy system initialized!");
    }

    // Economy API
    public double getBalance(UUID player) {
        return balances.getOrDefault(player, config.get().getStartingBalance());
    }

    public void setBalance(UUID player, double amount) {
        balances.put(player, Math.max(0, Math.min(amount, config.get().getMaxBalance())));
    }

    public boolean withdraw(UUID player, double amount) {
        double balance = getBalance(player);
        if (balance >= amount) {
            setBalance(player, balance - amount);
            return true;
        }
        return false;
    }

    public void deposit(UUID player, double amount) {
        setBalance(player, getBalance(player) + amount);
    }

    public boolean transfer(UUID from, UUID to, double amount) {
        if (withdraw(from, amount)) {
            deposit(to, amount);
            return true;
        }
        return false;
    }

    public String formatCurrency(double amount) {
        return config.get().getCurrencySymbol() + String.format("%.2f", amount);
    }

    public EconomyConfig getEconomyConfig() {
        return config.get();
    }
}

// Configuration
class EconomyConfig {
    @Nonnull
    public static final BuilderCodec<EconomyConfig> CODEC = BuilderCodec
        .builder(EconomyConfig.class, Builder::new)
        .append(new KeyedCodec<>("startingBalance", Codec.DOUBLE),
                (cfg, val) -> {}, EconomyConfig::getStartingBalance)
        .add()
        .append(new KeyedCodec<>("currencyName", Codec.STRING),
                (cfg, val) -> {}, EconomyConfig::getCurrencyName)
        .add()
        .append(new KeyedCodec<>("currencySymbol", Codec.STRING),
                (cfg, val) -> {}, EconomyConfig::getCurrencySymbol)
        .add()
        .append(new KeyedCodec<>("maxBalance", Codec.DOUBLE),
                (cfg, val) -> {}, EconomyConfig::getMaxBalance)
        .add()
        .build();

    private final double startingBalance;
    private final String currencyName;
    private final String currencySymbol;
    private final double maxBalance;

    private EconomyConfig(Builder builder) {
        this.startingBalance = builder.startingBalance;
        this.currencyName = builder.currencyName;
        this.currencySymbol = builder.currencySymbol;
        this.maxBalance = builder.maxBalance;
    }

    public double getStartingBalance() { return startingBalance; }
    public String getCurrencyName() { return currencyName; }
    public String getCurrencySymbol() { return currencySymbol; }
    public double getMaxBalance() { return maxBalance; }

    public static class Builder {
        private double startingBalance = 100.0;
        private String currencyName = "Coins";
        private String currencySymbol = "$";
        private double maxBalance = 1000000.0;

        public EconomyConfig build() {
            return new EconomyConfig(this);
        }
    }
}

// Balance command
class BalanceCommand extends AbstractPlayerCommand {

    public BalanceCommand() {
        super("balance", "server.commands.balance.desc");
        addAliases("bal", "money");
        requirePermission("economy.balance");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        double balance = EconomyPlugin.get().getBalance(sender.getUuid());
        context.sendMessage(Message.raw("Your balance: " + EconomyPlugin.get().formatCurrency(balance)));
    }
}

// Pay command
class PayCommand extends AbstractPlayerCommand {

    @Nonnull private final RequiredArg<UUID> targetArg;
    @Nonnull private final RequiredArg<Double> amountArg;

    public PayCommand() {
        super("pay", "server.commands.pay.desc");
        targetArg = withRequiredArg("player", "server.commands.pay.player.desc", ArgTypes.PLAYER_UUID);
        amountArg = withRequiredArg("amount", "server.commands.pay.amount.desc", ArgTypes.DOUBLE);
        requirePermission("economy.pay");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        UUID targetUuid = targetArg.get(context);
        double amount = amountArg.get(context);

        if (amount <= 0) {
            context.sendMessage(Message.raw("Amount must be positive!"));
            return;
        }

        EconomyPlugin economy = EconomyPlugin.get();
        if (economy.transfer(sender.getUuid(), targetUuid, amount)) {
            context.sendMessage(Message.raw("You paid " + economy.formatCurrency(amount)));
        } else {
            context.sendMessage(Message.raw("Insufficient funds!"));
        }
    }
}

// Admin economy command
class EcoCommand extends CommandBase {

    @Nonnull private final RequiredArg<String> actionArg;
    @Nonnull private final RequiredArg<UUID> targetArg;
    @Nonnull private final RequiredArg<Double> amountArg;

    public EcoCommand() {
        super("eco", "server.commands.eco.desc");
        actionArg = withRequiredArg("action", "server.commands.eco.action.desc", ArgTypes.STRING);
        targetArg = withRequiredArg("player", "server.commands.eco.player.desc", ArgTypes.PLAYER_UUID);
        amountArg = withRequiredArg("amount", "server.commands.eco.amount.desc", ArgTypes.DOUBLE);
        requirePermission("economy.admin");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context) {
        String action = actionArg.get(context).toLowerCase();
        UUID targetUuid = targetArg.get(context);
        double amount = amountArg.get(context);

        EconomyPlugin economy = EconomyPlugin.get();

        switch (action) {
            case "give":
                economy.deposit(targetUuid, amount);
                context.sendMessage(Message.raw("Gave " + economy.formatCurrency(amount)));
                break;
            case "take":
                economy.withdraw(targetUuid, amount);
                context.sendMessage(Message.raw("Took " + economy.formatCurrency(amount)));
                break;
            case "set":
                economy.setBalance(targetUuid, amount);
                context.sendMessage(Message.raw("Set balance to " + economy.formatCurrency(amount)));
                break;
            default:
                context.sendMessage(Message.raw("Unknown action: " + action + ". Use give/take/set"));
        }
    }
}
```

---

## Teleportation Plugin

Home and warp teleportation system.

### TeleportationPlugin.java

```java
package com.example.teleportation;

import com.hypixel.hytale.math.vector.Transform;
import com.hypixel.hytale.math.vector.Vector3d;
import com.hypixel.hytale.server.core.Message;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.command.system.CommandContext;
import com.hypixel.hytale.server.core.command.system.CommandRegistry;
import com.hypixel.hytale.server.core.command.system.basecommands.AbstractPlayerCommand;
import com.hypixel.hytale.server.core.command.system.arguments.system.RequiredArg;
import com.hypixel.hytale.server.core.command.system.arguments.system.DefaultArg;
import com.hypixel.hytale.server.core.command.system.arguments.types.ArgTypes;
import com.hypixel.hytale.server.core.universe.PlayerRef;
import com.hypixel.hytale.server.core.universe.Universe;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.logging.Level;
import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public class TeleportationPlugin extends JavaPlugin {

    private static TeleportationPlugin instance;

    // Player homes: UUID -> Map<homeName, Location>
    private final Map<UUID, Map<String, Location>> homes = new ConcurrentHashMap<>();

    // Server warps: warpName -> Location
    private final Map<String, Location> warps = new ConcurrentHashMap<>();

    // Teleport requests: targetUUID -> requesterUUID
    private final Map<UUID, TeleportRequest> teleportRequests = new ConcurrentHashMap<>();

    public TeleportationPlugin(@Nonnull JavaPluginInit init) {
        super(init);
    }

    @Nonnull
    public static TeleportationPlugin get() {
        return instance;
    }

    @Override
    protected void setup() {
        instance = this;

        CommandRegistry commandRegistry = getCommandRegistry();

        // Home commands
        commandRegistry.registerCommand(new SetHomeCommand());
        commandRegistry.registerCommand(new HomeCommand());
        commandRegistry.registerCommand(new DelHomeCommand());
        commandRegistry.registerCommand(new HomesCommand());

        // Warp commands
        commandRegistry.registerCommand(new SetWarpCommand());
        commandRegistry.registerCommand(new WarpCommand());
        commandRegistry.registerCommand(new WarpsCommand());

        // TPA commands
        commandRegistry.registerCommand(new TpaCommand());
        commandRegistry.registerCommand(new TpAcceptCommand());

        getLogger().at(Level.INFO).log("Teleportation system initialized!");
    }

    // Location data class
    public static class Location {
        public final String world;
        public final double x, y, z;
        public final float yaw, pitch;

        public Location(String world, double x, double y, double z, float yaw, float pitch) {
            this.world = world;
            this.x = x;
            this.y = y;
            this.z = z;
            this.yaw = yaw;
            this.pitch = pitch;
        }

        public static Location fromPlayerRef(PlayerRef playerRef) {
            Transform transform = playerRef.getTransform();
            Vector3d pos = transform.getPosition();
            return new Location(
                playerRef.getWorld().getName(),
                pos.x, pos.y, pos.z,
                transform.getRotation().getYaw(),
                transform.getRotation().getPitch()
            );
        }
    }

    // Teleport request
    public static class TeleportRequest {
        public final UUID requester;
        public final long timestamp;

        public TeleportRequest(UUID requester) {
            this.requester = requester;
            this.timestamp = System.currentTimeMillis();
        }

        public boolean isExpired() {
            return System.currentTimeMillis() - timestamp > 60000; // 60 seconds
        }
    }

    // Home management
    public void setHome(UUID player, String name, Location location) {
        homes.computeIfAbsent(player, k -> new ConcurrentHashMap<>()).put(name.toLowerCase(), location);
    }

    @Nullable
    public Location getHome(UUID player, String name) {
        Map<String, Location> playerHomes = homes.get(player);
        return playerHomes != null ? playerHomes.get(name.toLowerCase()) : null;
    }

    public boolean deleteHome(UUID player, String name) {
        Map<String, Location> playerHomes = homes.get(player);
        return playerHomes != null && playerHomes.remove(name.toLowerCase()) != null;
    }

    public Set<String> getHomeNames(UUID player) {
        Map<String, Location> playerHomes = homes.get(player);
        return playerHomes != null ? playerHomes.keySet() : Collections.emptySet();
    }

    // Warp management
    public void setWarp(String name, Location location) {
        warps.put(name.toLowerCase(), location);
    }

    @Nullable
    public Location getWarp(String name) {
        return warps.get(name.toLowerCase());
    }

    public Set<String> getWarpNames() {
        return warps.keySet();
    }

    // TPA management
    public void addTeleportRequest(UUID target, UUID requester) {
        teleportRequests.put(target, new TeleportRequest(requester));
    }

    @Nullable
    public TeleportRequest getTeleportRequest(UUID target) {
        TeleportRequest request = teleportRequests.get(target);
        if (request != null && request.isExpired()) {
            teleportRequests.remove(target);
            return null;
        }
        return request;
    }

    public void removeTeleportRequest(UUID target) {
        teleportRequests.remove(target);
    }
}

// Set home command
class SetHomeCommand extends AbstractPlayerCommand {

    @Nonnull
    private final DefaultArg<String> nameArg;

    public SetHomeCommand() {
        super("sethome", "server.commands.sethome.desc");
        nameArg = withDefaultArg("name", "server.commands.sethome.name.desc",
                                 ArgTypes.STRING, "home", "home");
        requirePermission("teleportation.sethome");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        String homeName = nameArg.get(context);
        TeleportationPlugin.Location location = TeleportationPlugin.Location.fromPlayerRef(sender);
        TeleportationPlugin.get().setHome(sender.getUuid(), homeName, location);
        context.sendMessage(Message.raw("Home '" + homeName + "' set!"));
    }
}

// Home command
class HomeCommand extends AbstractPlayerCommand {

    @Nonnull
    private final DefaultArg<String> nameArg;

    public HomeCommand() {
        super("home", "server.commands.home.desc");
        nameArg = withDefaultArg("name", "server.commands.home.name.desc",
                                 ArgTypes.STRING, "home", "home");
        requirePermission("teleportation.home");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        String homeName = nameArg.get(context);
        TeleportationPlugin.Location home = TeleportationPlugin.get().getHome(sender.getUuid(), homeName);

        if (home == null) {
            context.sendMessage(Message.raw("Home '" + homeName + "' not found!"));
            return;
        }

        // Teleport player using world.execute for thread safety
        sender.getWorld().execute(() -> {
            // Teleport implementation
        });

        context.sendMessage(Message.raw("Teleported to home '" + homeName + "'!"));
    }
}

// Delete home command
class DelHomeCommand extends AbstractPlayerCommand {

    @Nonnull
    private final RequiredArg<String> nameArg;

    public DelHomeCommand() {
        super("delhome", "server.commands.delhome.desc");
        nameArg = withRequiredArg("name", "server.commands.delhome.name.desc", ArgTypes.STRING);
        requirePermission("teleportation.delhome");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        String homeName = nameArg.get(context);

        if (TeleportationPlugin.get().deleteHome(sender.getUuid(), homeName)) {
            context.sendMessage(Message.raw("Home '" + homeName + "' deleted!"));
        } else {
            context.sendMessage(Message.raw("Home '" + homeName + "' not found!"));
        }
    }
}

// List homes command
class HomesCommand extends AbstractPlayerCommand {

    public HomesCommand() {
        super("homes", "server.commands.homes.desc");
        requirePermission("teleportation.homes");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        Set<String> homeNames = TeleportationPlugin.get().getHomeNames(sender.getUuid());

        if (homeNames.isEmpty()) {
            context.sendMessage(Message.raw("You have no homes set!"));
        } else {
            context.sendMessage(Message.raw("Your homes: " + String.join(", ", homeNames)));
        }
    }
}

// Set warp command
class SetWarpCommand extends AbstractPlayerCommand {

    @Nonnull
    private final RequiredArg<String> nameArg;

    public SetWarpCommand() {
        super("setwarp", "server.commands.setwarp.desc");
        nameArg = withRequiredArg("name", "server.commands.setwarp.name.desc", ArgTypes.STRING);
        requirePermission("teleportation.setwarp");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        String warpName = nameArg.get(context);
        TeleportationPlugin.Location location = TeleportationPlugin.Location.fromPlayerRef(sender);
        TeleportationPlugin.get().setWarp(warpName, location);
        context.sendMessage(Message.raw("Warp '" + warpName + "' set!"));
    }
}

// Warp command
class WarpCommand extends AbstractPlayerCommand {

    @Nonnull
    private final RequiredArg<String> nameArg;

    public WarpCommand() {
        super("warp", "server.commands.warp.desc");
        nameArg = withRequiredArg("name", "server.commands.warp.name.desc", ArgTypes.STRING);
        requirePermission("teleportation.warp");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        String warpName = nameArg.get(context);
        TeleportationPlugin.Location warp = TeleportationPlugin.get().getWarp(warpName);

        if (warp == null) {
            context.sendMessage(Message.raw("Warp '" + warpName + "' not found!"));
            return;
        }

        context.sendMessage(Message.raw("Teleported to warp '" + warpName + "'!"));
    }
}

// List warps command
class WarpsCommand extends AbstractPlayerCommand {

    public WarpsCommand() {
        super("warps", "server.commands.warps.desc");
        requirePermission("teleportation.warps");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        Set<String> warpNames = TeleportationPlugin.get().getWarpNames();

        if (warpNames.isEmpty()) {
            context.sendMessage(Message.raw("No warps available!"));
        } else {
            context.sendMessage(Message.raw("Warps: " + String.join(", ", warpNames)));
        }
    }
}

// TPA command
class TpaCommand extends AbstractPlayerCommand {

    @Nonnull
    private final RequiredArg<UUID> targetArg;

    public TpaCommand() {
        super("tpa", "server.commands.tpa.desc");
        targetArg = withRequiredArg("player", "server.commands.tpa.player.desc", ArgTypes.PLAYER_UUID);
        requirePermission("teleportation.tpa");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        UUID targetUuid = targetArg.get(context);
        TeleportationPlugin.get().addTeleportRequest(targetUuid, sender.getUuid());

        // Notify target player
        PlayerRef target = Universe.get().getPlayer(targetUuid);
        if (target != null) {
            target.sendMessage(Message.raw(sender.getUsername() + " wants to teleport to you. Use /tpaccept"));
        }

        context.sendMessage(Message.raw("Teleport request sent!"));
    }
}

// TPA accept command
class TpAcceptCommand extends AbstractPlayerCommand {

    public TpAcceptCommand() {
        super("tpaccept", "server.commands.tpaccept.desc");
        requirePermission("teleportation.tpaccept");
    }

    @Override
    protected void executeSync(@Nonnull CommandContext context, @Nonnull PlayerRef sender) {
        TeleportationPlugin.TeleportRequest request = TeleportationPlugin.get().getTeleportRequest(sender.getUuid());

        if (request == null) {
            context.sendMessage(Message.raw("You have no pending teleport requests!"));
            return;
        }

        TeleportationPlugin.get().removeTeleportRequest(sender.getUuid());
        context.sendMessage(Message.raw("Teleport request accepted!"));
    }
}
```

---

## See Also

- [Plugin System](Plugin-System) - Plugin architecture details
- [Creating Plugins](Creating-Plugins) - Step-by-step plugin creation guide
- [Event System](Event-System) - Event handling documentation
- [Command System](Command-System) - Command registration and handling
- [Component System](Component-System) - Entity Component System documentation
