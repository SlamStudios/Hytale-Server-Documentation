# Hytale Server Documentation

Welcome to the comprehensive documentation for the decompiled Hytale Server. This wiki provides detailed information about the server's architecture, systems, and modules.

## Overview

The Hytale Server is a sophisticated Java-based game server developed by Hypixel Studios. It features a modular plugin architecture, an Entity Component System (ECS) for game logic, and uses the QUIC/TCP protocols for networking.

### Key Features

- **Plugin-Based Architecture**: Modular design allowing features to be loaded as plugins
- **Entity Component System (ECS)**: Modern game architecture for entities and systems
- **Asset Store System**: Dynamic asset loading and hot-reloading capabilities
- **Procedural World Generation**: Advanced terrain and structure generation
- **QUIC + TCP Networking**: Modern network protocol support via Netty

## Quick Navigation

### Core Systems
- [Architecture Overview](Architecture)
- [Server Lifecycle](Server-Lifecycle)
- [Configuration](Configuration)

### Modules
- [Plugin System](Plugin-System)
- [Event System](Event-System)
- [Component System (ECS)](Component-System)
- [Networking & Protocol](Networking)
- [Entity System](Entity-System)
- [World & Universe](World-Universe)
- [Asset Store](Asset-Store)
- [Command System](Command-System)

### Built-in Plugins
- [World Generation](Built‐in-Plugins#worldgenplugin)
- [Block Physics](Built‐in-Plugins#blockphysicsplugin)
- [Crafting System](Built‐in-Plugins#craftingplugin)
- [NPC System](Built‐in-Plugins#npceditorplugin)
- [Weather System](Built‐in-Plugins#weatherplugin)
- [All Builtin Plugins](Built-in-Plugins.md)

### Development
- [Creating Plugins](Creating-Plugins)
- [Asset Types](Asset-Types)
- [Block Types](Block-Types.md)
- [Entity Types](Entity-Types)

## Technical Details

| Property | Value |
|----------|-------|
| Default Port | 5520 |
| Protocol | QUIC (primary), TCP (fallback) |
| Language | Java |
| Network Library | Netty |
| Configuration | JSON |

## Package Structure

```
com.hypixel.hytale
├── Main.java                    # Application entry point
├── LateMain.java               # Post-classloader initialization
├── assetstore/                 # Asset management system
├── builtin/                    # Built-in plugins and features
│   ├── adventure/              # Adventure mode features
│   ├── ambience/               # Ambience and atmosphere
│   ├── blockphysics/           # Block physics simulation
│   ├── crafting/               # Crafting system
│   ├── fluid/                  # Fluid dynamics
│   ├── npc*/                   # NPC-related modules
│   ├── weather/                # Weather system
│   └── worldgen/               # World generation
├── codec/                      # Serialization codecs
├── common/                     # Shared utilities
├── component/                  # ECS component system
├── event/                      # Event bus system
├── logger/                     # Logging framework
├── math/                       # Math utilities
├── metrics/                    # Performance metrics
├── procedural/                 # Procedural generation library
├── protocol/                   # Network protocol packets
├── registry/                   # Registry system
├── server/                     # Core server implementation
│   └── core/
│       ├── asset/              # Asset management
│       ├── auth/               # Authentication
│       ├── command/            # Command system
│       ├── console/            # Console interface
│       ├── entity/             # Entity management
│       ├── event/              # Server events
│       ├── io/                 # Network I/O
│       ├── modules/            # Core modules
│       ├── plugin/             # Plugin management
│       ├── universe/           # Universe/world management
│       └── util/               # Server utilities
└── storage/                    # Data persistence
```

## Getting Started

1. Review the [Architecture Overview](Architecture) to understand the system design
2. Learn about the [Plugin System](Plugin-System) for extending functionality
3. Explore the [Event System](Event-System) for hooking into server events
4. Check out [Builtin Plugins](Builtin-Plugins) for implementation examples

## Contributing

This documentation is generated from analysis of decompiled code. Contributions and corrections are welcome.
