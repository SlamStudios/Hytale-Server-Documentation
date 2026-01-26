
The Hytale Server includes a comprehensive command system for console and player interaction. This document covers command registration, execution, and creating custom commands.

## Overview

The command system is implemented in `com.hypixel.hytale.server.core.command` and provides:

- Console and player commands
- Permission-based access control
- Tab completion
- Command arguments parsing
- Asynchronous execution support

## Architecture

```
CommandManager
├── Command Registry
│   └── Registered Commands
├── Command Parser
│   └── Argument Parsing
├── Command Executor
│   └── Async/Sync Execution
└── Tab Completer
    └── Suggestions
```

## Key Classes

| Class | Location | Description |
|-------|----------|-------------|
| `CommandManager` | `server.core.command.system` | Central command management |
| `CommandRegistry` | `server.core.command.system` | Command registration |
| `CommandSender` | `server.core.command.system` | Command executor interface |
| `CommandOwner` | `server.core.command.system` | Command owner (plugin) |
| `ConsoleSender` | `server.core.console` | Console command sender |

## Command Manager

Central command handling:

```java
CommandManager manager = CommandManager.get();
// or
CommandManager manager = HytaleServer.get().getCommandManager();

// Execute command
manager.handleCommand(sender, "command args");

// Async execution
CompletableFuture<Void> future = manager.handleCommands(sender, commandQueue);
```

## Command Registration

### Via Plugin Registry

```java
public class MyPlugin extends JavaPlugin {
    @Override
    protected void setup() {
        CommandRegistry registry = getCommandRegistry();
        registry.register(new MyCommand());
    }
}
```

### Command Implementation

```java
public class MyCommand implements Command {
    @Override
    public String getName() {
        return "mycommand";
    }

    @Override
    public String[] getAliases() {
        return new String[]{"mc", "mycmd"};
    }

    @Override
    public String getDescription() {
        return "My custom command";
    }

    @Override
    public String getUsage() {
        return "/mycommand <arg1> [arg2]";
    }

    @Override
    public String getPermission() {
        return "myplugin.mycommand";
    }

    @Override
    public void execute(CommandSender sender, String[] args) {
        // Command logic
        sender.sendMessage("Hello!");
    }

    @Override
    public List<String> tabComplete(CommandSender sender, String[] args) {
        // Tab completion suggestions
        return List.of("option1", "option2");
    }
}
```

## Command Sender

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
```

### Player Sender

Players implement `CommandSender`:

```java
Player player = // ...
player.sendMessage("Player message");
if (player.hasPermission("some.permission")) {
    // Allowed
}
```

## Built-in Commands

### World Commands

| Command | Description |
|---------|-------------|
| `/world list` | List worlds |
| `/world add <name>` | Create world |
| `/world remove <name>` | Remove world |
| `/world load <name>` | Load world |
| `/world save [name]` | Save world |
| `/world setdefault <name>` | Set default |
| `/world prune` | Clean chunks |

### Block Commands

| Command | Description |
|---------|-------------|
| `/block get <x> <y> <z>` | Get block info |
| `/block set <x> <y> <z> <type>` | Set block |
| `/block getstate` | Get state |
| `/block setstate` | Set state |
| `/block select` | Selection mode |
| `/block bulk` | Bulk operations |
| `/block row` | Row operations |

### Teleport Commands

| Command | Description |
|---------|-------------|
| `/tp <player>` | Teleport to player |
| `/tp <x> <y> <z>` | Teleport to coords |
| `/tp <player> <target>` | Teleport player |

### Server Commands

| Command | Description |
|---------|-------------|
| `/bindings` | Network bindings |
| `/auth` | Authentication |

### Performance Commands

| Command | Description |
|---------|-------------|
| `/world tps` | Show TPS |
| `/world perf` | Performance stats |
| `/world perf graph` | Performance graph |

### World Config Commands

| Command | Description |
|---------|-------------|
| `/worldconfig` | World configuration |

## Subcommands

Commands can have subcommands:

```java
public class ParentCommand implements Command {
    private final Map<String, Command> subcommands = new HashMap<>();

    public ParentCommand() {
        subcommands.put("sub1", new Sub1Command());
        subcommands.put("sub2", new Sub2Command());
    }

    @Override
    public void execute(CommandSender sender, String[] args) {
        if (args.length == 0) {
            sendHelp(sender);
            return;
        }

        Command sub = subcommands.get(args[0].toLowerCase());
        if (sub != null) {
            String[] subArgs = Arrays.copyOfRange(args, 1, args.length);
            sub.execute(sender, subArgs);
        } else {
            sender.sendMessage("Unknown subcommand: " + args[0]);
        }
    }

    @Override
    public List<String> tabComplete(CommandSender sender, String[] args) {
        if (args.length == 1) {
            return subcommands.keySet().stream()
                .filter(s -> s.startsWith(args[0].toLowerCase()))
                .collect(Collectors.toList());
        }
        // Delegate to subcommand
        Command sub = subcommands.get(args[0].toLowerCase());
        if (sub != null) {
            return sub.tabComplete(sender,
                Arrays.copyOfRange(args, 1, args.length));
        }
        return Collections.emptyList();
    }
}
```

## Permissions

Commands check permissions:

```java
@Override
public void execute(CommandSender sender, String[] args) {
    // Automatic permission check
    // Or manual:
    if (!sender.hasPermission("extra.permission")) {
        sender.sendMessage("No permission!");
        return;
    }
    // ...
}
```

### Permission Format

```
<plugin>.<command>
<plugin>.<command>.<subcommand>
<plugin>.<feature>.<action>
```

## Messages

The `Message` class for formatted messages:

```java
Message message = Message.of("message.key")
    .with("param1", value1)
    .with("param2", value2);

sender.sendMessage(message);
```

### Message Components

```java
// Plain text
sender.sendMessage("Plain text message");

// Formatted
Message formatted = Message.of("formatted.key");

// With parameters
Message parameterized = Message.of("key")
    .with("player", playerName)
    .with("amount", 100);
```

## Async Commands

For long-running operations:

```java
@Override
public void execute(CommandSender sender, String[] args) {
    CompletableFuture.runAsync(() -> {
        // Long operation
        String result = expensiveOperation();

        // Send result on main thread
        sender.sendMessage("Result: " + result);
    });
}
```

## Command Macros

Macros for command sequences (MacroCommandPlugin):

```java
// Define macro
/macro define myMacro "cmd1; cmd2; cmd3"

// Execute macro
/macro run myMacro

// With parameters
/macro define greet "say Hello, {0}!"
/macro run greet Player1
```

## Tab Completion

Provide suggestions:

```java
@Override
public List<String> tabComplete(CommandSender sender, String[] args) {
    if (args.length == 1) {
        // First argument suggestions
        return List.of("option1", "option2", "option3")
            .stream()
            .filter(s -> s.startsWith(args[0].toLowerCase()))
            .collect(Collectors.toList());
    }
    if (args.length == 2) {
        // Second argument based on first
        if (args[0].equals("player")) {
            return getOnlinePlayerNames()
                .stream()
                .filter(s -> s.startsWith(args[1]))
                .collect(Collectors.toList());
        }
    }
    return Collections.emptyList();
}
```

## Name Matching

Fuzzy name matching utility:

```java
NameMatching matcher;

// Find matching names
List<String> matches = matcher.match(input, candidates);
```

## Best Practices

1. **Check permissions** - Always verify access
2. **Validate arguments** - Handle missing/invalid args
3. **Provide feedback** - Inform user of results
4. **Support tab completion** - Improve UX
5. **Use async** - For expensive operations
6. **Document usage** - Clear help messages
7. **Handle errors** - Graceful error messages

## Error Handling

```java
@Override
public void execute(CommandSender sender, String[] args) {
    try {
        // Command logic
    } catch (IllegalArgumentException e) {
        sender.sendMessage("Invalid argument: " + e.getMessage());
    } catch (Exception e) {
        sender.sendMessage("Error executing command");
        logger.error("Command error", e);
    }
}
```

## Boot Commands

Commands executed on server start:

```
java -jar server.jar --boot-command "world load main" --boot-command "say Server started"
```
