
The Hytale Server includes numerous built-in plugins that implement core game functionality. This document provides an overview of all built-in plugins.

## Plugin Categories

- [Adventure Plugins](#adventure-plugins) - RPG/adventure features
- [Core Plugins](#core-plugins) - Essential game systems
- [Movement Plugins](#movement-plugins) - Player movement mechanics
- [World Plugins](#world-plugins) - World generation and management
- [Editor Plugins](#editor-plugins) - Development tools

## Adventure Plugins

Located in `com.hypixel.hytale.builtin.adventure`:

### CameraPlugin
Camera effects and shake system.
- Camera shake effects
- View bobbing
- Camera positioning

### FarmingPlugin
Crop farming and agriculture system.
- Crop growth stages
- Soil management
- Fertilizers and water
- Coops and animal farming
- Capture crates

### MemoriesPlugin
NPC memory and knowledge system.
- NPCs remember player interactions
- Knowledge persistence

### ObjectivePlugin
Quest objective tracking.
- Objective tracking
- Progress updates
- Completion handling

### ReputationPlugin
Reputation system for NPCs and factions.
- Reputation levels
- Faction relationships
- Reputation effects

### ShopPlugin
Trading and shop system.
- Buy/sell mechanics
- Shop UI
- Pricing

### NPCShopPlugin
NPC-specific shop functionality.
- NPC shopkeepers
- Dynamic inventory

### NPCObjectivesPlugin
NPC objective integration.
- NPC quest givers
- Objective NPCs

### NPCReputationPlugin
NPC reputation integration.
- Per-NPC reputation
- Reputation effects on NPCs

### ObjectiveReputationPlugin
Reputation from objectives.
- Reputation rewards
- Objective-based reputation

### ObjectiveShopPlugin
Shop-objective integration.
- Shop-based objectives
- Purchase quests

### ShopReputationPlugin
Reputation effects on shops.
- Reputation-based pricing
- Shop access control

### StashPlugin
Item stash/storage system.
- Personal storage
- Stash UI

### TeleporterPlugin
Teleportation waypoints.
- Teleporter blocks
- Fast travel network

### WorldLocationConditionPlugin
Location-based conditions.
- Area triggers
- Location requirements

## Core Plugins

### AmbiencePlugin
`com.hypixel.hytale.builtin.ambience`

Ambient sounds and atmospheric effects.
- Ambient sound playback
- Environmental audio
- Music triggers

### AssetEditorPlugin
`com.hypixel.hytale.builtin.asseteditor`

In-game asset editing tools.
- Real-time asset modification
- JSON editor
- Live preview

### BedsPlugin
`com.hypixel.hytale.builtin.beds`

Bed mechanics and spawn points.
- Sleep mechanics
- Spawn point setting
- Time skipping (multiplayer)

### BlockPhysicsPlugin
`com.hypixel.hytale.builtin.blockphysics`

Block physics simulation.
- Gravity for blocks
- Structural support
- Physics-based destruction

### BlockSpawnerPlugin
`com.hypixel.hytale.builtin.blockspawner`

Mob spawner blocks.
- Entity spawning
- Spawn conditions
- Rate limiting

### BlockTickPlugin
`com.hypixel.hytale.builtin.blocktick`

Block tick scheduling.
- Random block ticks
- Scheduled updates
- Tick priorities

### BuilderToolsPlugin
`com.hypixel.hytale.builtin.buildertools`

World editing tools.
- Selection tools
- Fill/replace
- Copy/paste
- World editing commands

### MacroCommandPlugin
`com.hypixel.hytale.builtin.commandmacro`

Command macro system.
- Command sequences
- Macro definitions
- Parameter substitution

### CraftingPlugin
`com.hypixel.hytale.builtin.crafting`

Crafting system.
- Recipe definitions
- Crafting benches
- Processing benches
- Diagram crafting

### CreativeHubPlugin
`com.hypixel.hytale.builtin.creativehub`

Creative mode hub functionality.
- Creative inventory
- Build mode features

### DeployablesPlugin
`com.hypixel.hytale.builtin.deployables`

Deployable items/structures.
- Placed items
- Temporary structures
- Deployable mechanics

### FluidPlugin
`com.hypixel.hytale.builtin.fluid`

Fluid simulation (water, lava).
- Fluid flow
- Fluid levels
- Fluid interactions

### InstancesPlugin
`com.hypixel.hytale.builtin.instances`

Dungeon/instance system.
- Instance creation
- Instance management
- Player isolation

### LANDiscoveryPlugin
`com.hypixel.hytale.builtin.landiscovery`

LAN server discovery.
- Broadcast server
- Discover servers
- Local multiplayer

### ModelPlugin
`com.hypixel.hytale.builtin.model`

Entity model system.
- Model loading
- Model rendering data
- Animation integration

### MountPlugin
`com.hypixel.hytale.builtin.mounts`

Mount system.
- Mountable entities
- Mount controls
- Mount abilities

### NPCEditorPlugin
`com.hypixel.hytale.builtin.npceditor`

NPC editing tools.
- NPC creation
- Behavior editing
- Dialogue editing

### NPCCombatActionEvaluatorPlugin
`com.hypixel.hytale.builtin.npccombatactionevaluator`

NPC combat AI.
- Combat decisions
- Action evaluation
- Threat assessment

### PathPlugin
`com.hypixel.hytale.builtin.path`

Pathfinding system.
- A* pathfinding
- Navigation mesh
- Path optimization

### PortalsPlugin
`com.hypixel.hytale.builtin.portals`

Portal system.
- Portal creation
- Dimension travel
- Portal linking

### TagSetPlugin
`com.hypixel.hytale.builtin.tagset`

Entity tagging system.
- Tag definitions
- Tag queries
- Tag-based logic

### TeleportPlugin
`com.hypixel.hytale.builtin.teleport`

Teleportation commands.
- /tp command
- Teleport requests
- Location teleport

### WeatherPlugin
`com.hypixel.hytale.builtin.weather`

Weather system.
- Weather types
- Weather transitions
- Weather effects

### WorldGenPlugin
`com.hypixel.hytale.builtin.worldgen`

World generation.
- Terrain generation
- Biome placement
- Structure generation

## Movement Plugins

### CrouchSlidePlugin
`com.hypixel.hytale.builtin.crouchslide`

Crouch sliding mechanics.
- Slide activation
- Slide physics
- Slide attacks

### MantlingPlugin
`com.hypixel.hytale.builtin.mantling`

Ledge climbing/mantling.
- Edge detection
- Climb animation
- Mantle mechanics

### ParkourPlugin
`com.hypixel.hytale.builtin.parkour`

Parkour movement.
- Wall running
- Special jumps
- Parkour mechanics

### SafetyRollPlugin
`com.hypixel.hytale.builtin.safetyroll`

Fall damage reduction.
- Roll mechanics
- Damage mitigation
- Timing system

### SprintForcePlugin
`com.hypixel.hytale.builtin.sprintforce`

Sprint mechanics.
- Sprint activation
- Speed boost
- Stamina (if applicable)

## Plugin Dependencies

Many plugins have dependencies on other plugins:

```
CraftingPlugin
├── Requires: BlockTickPlugin
└── Requires: AssetStore

FarmingPlugin
├── Requires: BlockTickPlugin
├── Requires: WeatherPlugin
└── Optional: CraftingPlugin

NPCShopPlugin
├── Requires: ShopPlugin
└── Requires: NPCPlugin

WeatherPlugin
├── Requires: AmbiencePlugin
└── Affects: FarmingPlugin
```

## Plugin Configuration

Plugins can be configured in `server-config.json`:

```json
{
    "modules": {
        "weather": {
            "enabled": true,
            "defaultWeather": "clear",
            "transitionDuration": 60
        },
        "crafting": {
            "enabled": true,
            "instantCraft": false
        }
    }
}
```

## Disabling Plugins

Plugins can be disabled via configuration:

```json
{
    "modules": {
        "parkour": {
            "enabled": false
        }
    }
}
```

## Plugin Load Order

Plugins are loaded in dependency order:

1. Core systems (EventBus, Registry)
2. Base plugins (BlockTick, Model)
3. Feature plugins (Crafting, Weather)
4. Integration plugins (NPCShop, ObjectiveReputation)

## Creating Custom Plugins

See [Plugin System](Plugin-System) and [Creating Plugins](Creating-Plugins) for information on creating custom plugins that integrate with built-in systems.
