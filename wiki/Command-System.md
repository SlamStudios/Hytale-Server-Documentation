# Command System

The Hytale Server includes a comprehensive command system for console and player interaction. This document covers command registration, execution, and creating custom commands.

## Overview

The command system is implemented in `com.hypixel.hytale.server.core.command` and provides:

- Console and player commands
- Permission-based access control
- Tab completion with suggestions
- Typed argument parsing with validation
- Subcommands and command variants
- Asynchronous execution support

## Architecture

```
CommandManager
├── CommandRegistry
│   └── Registered AbstractCommands
├── Argument System
│   ├── RequiredArg
│   ├── OptionalArg
│   ├── DefaultArg
│   └── FlagArg
├── ArgumentTypes
│   ├── SingleArgumentType
│   ├── ListArgumentType
│   └── Custom types
└── SuggestionProvider
    └── Tab completion
```

## Key Classes

| Class | Location | Description |
|-------|----------|-------------|
| `CommandManager` | `server.core.command.system` | Central command management |
| `CommandRegistry` | `server.core.command.system` | Command registration |
| `AbstractCommand` | `server.core.command.system` | Base class for all commands |
| `CommandContext` | `server.core.command.system` | Execution context with parsed arguments |
| `CommandSender` | `server.core.command.system` | Command executor interface |
| `ArgumentType<T>` | `server.core.command.system.arguments.types` | Argument type definition |

---

## Creating Commands

Commands extend `AbstractCommand` and define their arguments in the constructor.

### Basic Command Structure

```java
public class GreetCommand extends AbstractCommand {
    
    // Define arguments as fields
    private final RequiredArg<String> nameArg;
    private final OptionalArg<Integer> countArg;
    private final FlagArg loudFlag;
    
    public GreetCommand() {
        // Constructor: (name, description) or (name, description, requiresConfirmation)
        super("greet", "server.commands.greet.description");
        
        // Register required argument
        // withRequiredArg(name, description, argumentType)
        this.nameArg = withRequiredArg("name", 
            "server.commands.greet.args.name", 
            ArgTypes.STRING);
        
        // Register optional argument
        // withOptionalArg(name, description, argumentType)
        this.countArg = withOptionalArg("count", 
            "server.commands.greet.args.count", 
            ArgTypes.POSITIVE_INTEGER);
        
        // Register flag (boolean toggle)
        // withFlagArg(name, description)
        this.loudFlag = withFlagArg("loud", 
            "server.commands.greet.args.loud");
    }
    
    @Override
    @Nullable
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        // Get argument values from context
        String name = context.get(nameArg);
        Integer count = context.get(countArg);  // null if not provided
        boolean loud = context.provided(loudFlag);
        
        // Default value handling
        int times = (count != null) ? count : 1;
        
        // Build message
        String greeting = loud ? "HELLO " + name.toUpperCase() + "!" : "Hello " + name + "!";
        
        for (int i = 0; i < times; i++) {
            context.sendMessage(Message.raw(greeting));
        }
        
        return null;  // Return CompletableFuture for async, null for sync
    }
}
```

### Registering Commands

```java
public class MyPlugin extends JavaPlugin {
    @Override
    protected void setup() {
        CommandRegistry registry = getCommandRegistry();
        registry.register(new GreetCommand());
    }
}
```

---

## Argument Types

### Built-in Argument Types

The `ArgTypes` class provides common argument types:

| Type | Description | Example Input |
|------|-------------|---------------|
| `ArgTypes.STRING` | Any string | `hello` |
| `ArgTypes.INTEGER` | Any integer | `42`, `-5` |
| `ArgTypes.POSITIVE_INTEGER` | Integer > 0 | `1`, `100` |
| `ArgTypes.NON_NEGATIVE_INTEGER` | Integer >= 0 | `0`, `50` |
| `ArgTypes.FLOAT` | Decimal number | `3.14` |
| `ArgTypes.DOUBLE` | Double precision | `3.14159` |
| `ArgTypes.BOOLEAN` | true/false | `true`, `false` |
| `ArgTypes.PLAYER` | Online player | `PlayerName` |
| `ArgTypes.WORLD` | World name | `overworld` |
| `ArgTypes.BLOCK_TYPE` | Block type ID | `stone` |
| `ArgTypes.ITEM_TYPE` | Item type ID | `sword` |
| `ArgTypes.ENTITY_TYPE` | Entity type ID | `zombie` |
| `ArgTypes.UUID` | UUID | `550e8400-e29b-...` |
| `ArgTypes.GAME_MODE` | Game mode | `creative` |

### Relative/Coordinate Types

| Type | Description | Example Input |
|------|-------------|---------------|
| `ArgTypes.RELATIVE_INT` | Relative integer | `~5`, `~-3`, `10` |
| `ArgTypes.RELATIVE_FLOAT` | Relative float | `~0.5`, `3.14` |
| `ArgTypes.RELATIVE_INT_POSITION` | XYZ position | `~0 ~1 ~0` |
| `ArgTypes.RELATIVE_DOUBLE_POSITION` | XYZ position (double) | `~0.5 64 ~0.5` |
| `ArgTypes.RELATIVE_CHUNK_POSITION` | Chunk XZ | `~0 ~0` |

### Using Argument Types

```java
public class TeleportCommand extends AbstractCommand {
    private final RequiredArg<RelativeDoublePosition> positionArg;
    private final OptionalArg<Player> targetArg;
    
    public TeleportCommand() {
        super("tp", "server.commands.tp.description");
        
        // Position requires 3 parameters (x, y, z)
        this.positionArg = withRequiredArg("position",
            "server.commands.tp.args.position",
            ArgTypes.RELATIVE_DOUBLE_POSITION);
        
        // Optional player target
        this.targetArg = withOptionalArg("target",
            "server.commands.tp.args.target",
            ArgTypes.PLAYER);
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        RelativeDoublePosition pos = context.get(positionArg);
        Player target = context.get(targetArg);
        
        // If no target specified, use sender if they're a player
        if (target == null) {
            if (!context.isPlayer()) {
                context.sendMessage(Message.translation("server.commands.error.requiresPlayer"));
                return null;
            }
            target = context.senderAs(Player.class);
        }
        
        // Resolve relative position based on current location
        Vector3d currentPos = target.getPosition();
        Vector3d newPos = pos.resolve(currentPos);
        
        target.teleport(newPos);
        return null;
    }
}
```

---

## Argument Categories

### RequiredArg

Arguments that must be provided:

```java
// Single value
RequiredArg<String> nameArg = withRequiredArg("name", "description", ArgTypes.STRING);

// List of values (variable length)
RequiredArg<List<String>> namesArg = withListRequiredArg("names", "description", ArgTypes.STRING);
```

### OptionalArg

Arguments that can be omitted (prefixed with `--` when used):

```java
// Usage: /cmd --radius 10
OptionalArg<Integer> radiusArg = withOptionalArg("radius", "description", ArgTypes.POSITIVE_INTEGER);

// List version
OptionalArg<List<Integer>> valuesArg = withListOptionalArg("values", "description", ArgTypes.INTEGER);
```

### DefaultArg

Optional arguments with a default value:

```java
// Usage: /cmd --count 5  (defaults to 1 if not specified)
DefaultArg<Integer> countArg = withDefaultArg("count", "description", 
    ArgTypes.POSITIVE_INTEGER, 
    1,                          // default value
    "server.commands.default.one");  // default value description key
```

### FlagArg

Boolean flags (present = true, absent = false):

```java
// Usage: /cmd --silent
FlagArg silentFlag = withFlagArg("silent", "description");

// Check if provided
boolean isSilent = context.provided(silentFlag);
```

---

## Subcommands

Commands can have subcommands for hierarchical organization:

```java
public class WorldCommand extends AbstractCommand {
    
    public WorldCommand() {
        super("world", "server.commands.world.description");
        
        // Add subcommands
        addSubCommand(new WorldListCommand());
        addSubCommand(new WorldCreateCommand());
        addSubCommand(new WorldDeleteCommand());
        addSubCommand(new WorldTeleportCommand());
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        // Called when no subcommand matches - show help
        context.sendMessage(getUsageString(context.sender()));
        return null;
    }
}

public class WorldListCommand extends AbstractCommand {
    
    public WorldListCommand() {
        super("list", "server.commands.world.list.description");
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        // List all worlds
        Universe.get().getWorlds().forEach((name, world) -> {
            context.sendMessage(Message.raw("- " + name));
        });
        return null;
    }
}
```

Usage: `/world list`, `/world create myworld`, etc.

---

## Command Variants

For commands with different argument patterns:

```java
public class GiveCommand extends AbstractCommand {
    private final RequiredArg<Player> playerArg;
    private final RequiredArg<ItemType> itemArg;
    private final OptionalArg<Integer> amountArg;
    
    public GiveCommand() {
        super("give", "server.commands.give.description");
        
        this.playerArg = withRequiredArg("player", "desc", ArgTypes.PLAYER);
        this.itemArg = withRequiredArg("item", "desc", ArgTypes.ITEM_TYPE);
        this.amountArg = withOptionalArg("amount", "desc", ArgTypes.POSITIVE_INTEGER);
        
        // Add variant: /give <item> (gives to self)
        addUsageVariant(new GiveSelfVariant());
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        Player player = context.get(playerArg);
        ItemType item = context.get(itemArg);
        Integer amount = context.get(amountArg);
        // ... give item to player
        return null;
    }
}

// Variant with fewer required args
public class GiveSelfVariant extends AbstractCommand {
    private final RequiredArg<ItemType> itemArg;
    
    public GiveSelfVariant() {
        // Variants use description-only constructor (no name)
        super("server.commands.give.self.description");
        
        this.itemArg = withRequiredArg("item", "desc", ArgTypes.ITEM_TYPE);
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        if (!context.isPlayer()) {
            context.sendMessage(Message.translation("server.commands.error.requiresPlayer"));
            return null;
        }
        Player player = context.senderAs(Player.class);
        ItemType item = context.get(itemArg);
        // ... give item to self
        return null;
    }
}
```

---

## CommandContext

The `CommandContext` provides access to parsed arguments and sender info:

```java
@Override
protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
    // Get argument values (type-safe)
    String value = context.get(stringArg);
    Integer number = context.get(intArg);  // null if optional and not provided
    
    // Check if optional argument was provided
    if (context.provided(optionalArg)) {
        // Argument was explicitly provided
    }
    
    // Get raw input strings for an argument
    String[] rawInput = context.getInput(someArg);
    
    // Access sender
    CommandSender sender = context.sender();
    
    // Check if sender is a player
    if (context.isPlayer()) {
        Player player = context.senderAs(Player.class);
        Ref<EntityStore> playerRef = context.senderAsPlayerRef();
    }
    
    // Send messages
    context.sendMessage(Message.raw("Hello!"));
    context.sendMessage(Message.translation("some.translation.key")
        .param("name", value));
    
    // Get the original input string
    String input = context.getInputString();
    
    // Get the command that was called
    AbstractCommand cmd = context.getCalledCommand();
    
    return null;
}
```

---

## CommandSender

Interface for command executors:

```java
public interface CommandSender {
    void sendMessage(String message);
    void sendMessage(Message message);
    boolean hasPermission(String permission);
    String getName();
}
```

### Console Sender

```java
ConsoleSender console = ConsoleSender.INSTANCE;
console.sendMessage("Server message");
console.hasPermission("any.permission");  // Always true for console
```

### Player as Sender

Players implement `CommandSender`:

```java
Player player = // ...
player.sendMessage("Player message");

if (player.hasPermission("myplugin.admin")) {
    // Has permission
}
```

---

## Permissions

### Automatic Permission Generation

Permissions are auto-generated based on plugin and command name:

```java
// For a command "give" in plugin "myplugin" with group "com.example":
// Permission: com.example.command.give

// For subcommand "list" under "world":
// Permission: com.example.command.world.list
```

### Manual Permission

```java
public class AdminCommand extends AbstractCommand {
    public AdminCommand() {
        super("admin", "description");
        
        // Override auto-generated permission
        requirePermission("myplugin.admin.superuser");
    }
}
```

### Permission Groups

Associate commands with game modes:

```java
public class CreativeCommand extends AbstractCommand {
    public CreativeCommand() {
        super("fly", "description");
        
        // Only available in creative mode
        setPermissionGroup(GameMode.CREATIVE);
        
        // Or multiple groups
        setPermissionGroups("admin", "moderator");
    }
}
```

### Checking Permissions in Execute

```java
@Override
protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
    // Additional permission check
    if (!context.sender().hasPermission("extra.permission")) {
        context.sendMessage(Message.translation("server.commands.error.noPermission"));
        return null;
    }
    // ...
    return null;
}
```

---

## Confirmation Required

For dangerous commands that require `--confirm`:

```java
public class DeleteWorldCommand extends AbstractCommand {
    private final RequiredArg<World> worldArg;
    
    public DeleteWorldCommand() {
        // Third parameter = requiresConfirmation
        super("delete", "server.commands.world.delete.description", true);
        
        this.worldArg = withRequiredArg("world", "desc", ArgTypes.WORLD);
    }
    
    @Override
    protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
        // This only runs if --confirm was provided
        World world = context.get(worldArg);
        // ... delete world
        return null;
    }
}
```

Usage: `/world delete myworld --confirm`

---

## Async Execution

For long-running operations:

```java
@Override
protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
    return CompletableFuture.runAsync(() -> {
        // Long operation runs on separate thread
        String result = performExpensiveOperation();
        
        // Send result back to sender
        context.sendMessage(Message.raw("Result: " + result));
    });
}
```

---

## Tab Completion

Arguments automatically provide suggestions via `SuggestionProvider`. Custom argument types can override:

```java
public class MyArgumentType extends SingleArgumentType<MyType> {
    
    public MyArgumentType() {
        super("mytype", "usage", "example1", "example2");
    }
    
    @Override
    public void suggest(@Nonnull CommandSender sender, 
                        @Nonnull String textAlreadyEntered, 
                        int numParametersTyped, 
                        @Nonnull SuggestionResult result) {
        // Add suggestions based on what user has typed
        for (String option : getOptions()) {
            if (option.toLowerCase().startsWith(textAlreadyEntered.toLowerCase())) {
                result.suggest(option);
            }
        }
    }
    
    @Override
    @Nullable
    public MyType parse(@Nonnull String[] args, @Nonnull ParseResult result) {
        // Parse the argument
        MyType value = MyType.fromString(args[0]);
        if (value == null) {
            result.fail(Message.translation("error.invalid"));
            return null;
        }
        return value;
    }
}
```

---

## Built-in Commands

### World Commands

| Command | Description |
|---------|-------------|
| `/world list` | List all worlds |
| `/world add <name>` | Create new world |
| `/world remove <name>` | Remove world |
| `/world load <name>` | Load world |
| `/world save [name]` | Save world(s) |
| `/world setdefault <name>` | Set default world |
| `/world prune` | Remove empty chunks |
| `/world tps` | Show TPS |
| `/world perf` | Performance stats |

### Block Commands

| Command | Description |
|---------|-------------|
| `/block get <x> <y> <z>` | Get block info |
| `/block set <x> <y> <z> <type>` | Set block |
| `/block getstate` | Get block state |
| `/block setstate` | Set block state |

### Teleport Commands

| Command | Description |
|---------|-------------|
| `/tp <position>` | Teleport to coordinates |
| `/tp <player>` | Teleport to player |
| `/tp <player> <target>` | Teleport player to target |

---

## Messages

The `Message` class for formatted/translatable messages:

```java
// Raw text
Message.raw("Hello, World!")

// Translation key
Message.translation("server.commands.greet.success")

// With parameters
Message.translation("server.commands.greet.success")
    .param("name", playerName)
    .param("count", 5)

// Chaining
Message.raw("Result: ")
    .insert(Message.translation("status.success"))
    .insert("\n")
    .insert("Details: " + details)
```

---

## Error Handling

```java
@Override
protected CompletableFuture<Void> execute(@Nonnull CommandContext context) {
    try {
        // Command logic
        performAction();
        context.sendMessage(Message.translation("command.success"));
    } catch (IllegalArgumentException e) {
        context.sendMessage(Message.translation("command.error.invalidArg")
            .param("message", e.getMessage()));
    } catch (Exception e) {
        context.sendMessage(Message.translation("command.error.generic"));
        getLogger().at(Level.SEVERE).withCause(e).log("Command execution failed");
    }
    return null;
}
```

---

## Single Player Restriction

Disable commands in singleplayer:

```java
public class MultiplayerOnlyCommand extends AbstractCommand {
    public MultiplayerOnlyCommand() {
        super("serverinfo", "description");
        setUnavailableInSingleplayer(true);
    }
}
```

---

## Best Practices

1. **Use typed arguments** - Leverage `ArgTypes` for automatic parsing and validation
2. **Provide descriptions** - Use translation keys for all user-facing text
3. **Handle null optionals** - Always check if optional arguments were provided
4. **Use subcommands** - Organize complex commands hierarchically
5. **Require confirmation** - For destructive operations, use `requiresConfirmation`
6. **Check permissions** - Use auto-generated permissions or set explicit ones
7. **Return async futures** - For long operations, return `CompletableFuture`
8. **Validate in argument types** - Put parsing logic in custom `ArgumentType` classes
