# Hytale Server Documentation

Comprehensive documentation for the decompiled **Hytale Server**, covering architecture, systems, and internal modules. This repository serves as a technical reference for developers exploring how the Hytale server is structured and how its core systems operate.

> **Disclaimer**: This documentation is based on analysis of decompiled server code. It is intended for educational and research purposes.

---

## Overview

The Hytale Server is a Java-based game server developed by **Hypixel Studios**. It uses a modular, plugin-driven architecture, an **Entity Component System (ECS)** for game logic, and modern networking via **QUIC** with **TCP fallback**.

### Key Features

- **Plugin-Based Architecture** – Modular systems loaded as plugins
- **Entity Component System (ECS)** – Data-driven entity and system design
- **Asset Store System** – Dynamic asset loading and hot-reloading
- **Procedural World Generation** – Terrain, structures, and biome generation
- **QUIC + TCP Networking** – High-performance networking via Netty

---

## Documentation Navigation

### Core Systems

- [Architecture Overview](Architecture)
- [Server Lifecycle](Server-Lifecycle)
- [Configuration](Configuration)

### Modules

- [Plugin System](Plugin-System)
- [Event System](Event-System)
- [Component System (ECS)](Component-System-(ECS))
- [Networking & Protocol](Networking-&-Protocol)
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
- [All Built-in Plugins](Built-in-Plugins)

### Development

- [Creating Plugins](Creating-Plugins)
- [Asset Types](Asset-Types)
- [Block Types](Block-Types)
- [Entity Types](Entity-Types)

---

## Technical Details

| Property | Value |
|---------|-------|
| Default Port | 5520 |
| Protocol | QUIC (primary), TCP (fallback) |
| Language | Java |
| Network Library | Netty |
| Configuration Format | JSON |

---

## Package Structure

```text
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

---

## Getting Started

1. Read the [Architecture Overview](Architecture) to understand the overall design
2. Review the [Plugin System](Plugin-System) to learn how functionality is extended
3. Explore the [Event System](Event-System) to see how server events are dispatched
4. Inspect [Built-in Plugins](Built-in-Plugins) for real-world implementation examples

---

## Contributing

Contributions, corrections, and clarifications are welcome. If you discover inaccuracies or have additional insights from further analysis, feel free to open a pull request or issue.

---

## License

This repository contains documentation only. All original code, names, and assets related to Hytale are the property of Hypixel Studios.

