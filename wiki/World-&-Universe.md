
The Hytale Server organizes game content into a hierarchical structure with Universe at the top containing multiple Worlds. This document covers world management, chunks, and storage.

## Overview

The world system is implemented in `com.hypixel.hytale.server.core.universe` and provides:

- Multi-world support
- Chunk-based world storage
- Block and entity management
- Procedural generation integration
- Data persistence

## Hierarchy

```
Universe (Server)
├── World (Dimension/Realm)
│   ├── ChunkColumn
│   │   ├── WorldChunk (metadata)
│   │   ├── BlockChunk (blocks)
│   │   ├── EntityChunk (entities)
│   │   └── BlockComponentChunk (block components)
│   ├── WorldConfig
│   └── WorldGenerator
└── DataStoreProvider
    └── DiskDataStoreProvider
```

## Key Classes

### Universe

| Class | Location | Description |
|-------|----------|-------------|
| `Universe` | `server.core.universe` | Top-level container |
| `World` | `server.core.universe.world` | Individual world/dimension |
| `WorldConfig` | `server.core.universe.world` | World configuration |

### Chunks

| Class | Description |
|-------|-------------|
| `ChunkColumn` | Vertical stack of chunks |
| `WorldChunk` | Chunk metadata and state |
| `BlockChunk` | Block data storage |
| `EntityChunk` | Entity storage per chunk |
| `BlockComponentChunk` | Block component data |
| `ChunkSection` | 16x16x16 block section |

### Storage

| Class | Description |
|-------|-------------|
| `DataStoreProvider` | Storage abstraction |
| `DiskDataStoreProvider` | Disk-based storage |
| `DataStore` | Individual data store |
| `ChunkStore` | Chunk persistence |
| `EntityStore` | Entity persistence |

## Universe

The Universe is the top-level container for all worlds.

### Properties

```java
Universe universe = Universe.get();

// Get worlds
Map<String, World> worlds = universe.getWorlds();

// Get default world
World defaultWorld = universe.getDefaultWorld();

// Player count
int playerCount = universe.getPlayerCount();

// Universe path
Path path = universe.getPath();
```

### Lifecycle

```java
// Universe ready event
universe.getUniverseReady().join();

// Access after ready
World world = universe.getWorld("overworld");
```

## World

Each World represents a separate dimension or realm.

### World Properties

```java
World world = universe.getWorld("worldName");

// Basic info
String name = world.getName();
HytaleLogger logger = world.getLogger();

// Chunk access
ChunkColumn column = world.getChunkColumn(chunkX, chunkZ);
```

### World Configuration

```java
WorldConfig config = world.getConfig();
// Generator settings
// Spawn position
// World rules
```

## Chunks

### Chunk Coordinate System

```
World Coordinates: (x, y, z) in blocks
Chunk Coordinates: (chunkX, chunkZ) = (x >> 4, z >> 4)
Section Coordinates: sectionY = (y >> 4)
Local Coordinates: (x & 15, y & 15, z & 15)
```

### ChunkColumn

Vertical stack of chunk data:

```java
ChunkColumn column = world.getChunkColumn(chunkX, chunkZ);

// Access chunks
WorldChunk worldChunk = column.getWorldChunk();
BlockChunk blockChunk = column.getBlockChunk();
EntityChunk entityChunk = column.getEntityChunk();
```

### BlockChunk

Stores block data in sections:

```java
BlockChunk chunk = column.getBlockChunk();

// Get section
BlockSection section = chunk.getSection(sectionY);

// Block operations
BlockType type = section.getBlockType(localX, localY, localZ);
section.setBlockType(localX, localY, localZ, newType);
```

### ChunkSection

16x16x16 block storage with palette compression:

```java
BlockSection section;

// Block type access
int blockId = section.getBlockId(x, y, z);
section.setBlockId(x, y, z, blockId);

// Block state
int state = section.getBlockState(x, y, z);
section.setBlockState(x, y, z, state);
```

## Palette System

Efficient block storage using palettes:

```
ISectionPalette
├── EmptySectionPalette (all same block)
├── HalfByteSectionPalette (up to 16 types)
├── ByteSectionPalette (up to 256 types)
└── ShortSectionPalette (unlimited types)
```

### Palette Types

| Type | Bits/Block | Max Types |
|------|------------|-----------|
| Empty | 0 | 1 |
| HalfByte | 4 | 16 |
| Byte | 8 | 256 |
| Short | 16 | 65536 |

## Block Operations

### Block Accessors

```java
// Basic block access
BlockAccessor accessor = world.getBlockAccessor();
BlockType type = accessor.getBlockType(x, y, z);
accessor.setBlockType(x, y, z, newType);

// Chunk accessor for bulk operations
ChunkAccessor chunkAccessor = world.getChunkAccessor();
```

### Accessor Types

| Class | Description |
|-------|-------------|
| `BlockAccessor` | Direct block access |
| `ChunkAccessor` | Chunk-level access |
| `IChunkAccessorSync` | Synchronized access |
| `LocalCachedChunkAccessor` | Cached for performance |
| `OverridableChunkAccessor` | With fallback support |

## Block State

Blocks can have additional state data:

```java
// Block state registry
BlockStateRegistry registry = plugin.getBlockStateRegistry();

// Register state type
registry.register(MyBlockState.class, MyBlockState.CODEC);

// Get/set state
MyBlockState state = block.getState(MyBlockState.class);
block.setState(newState);
```

### Tickable Block State

```java
public class TickableBlockState {
    // State that receives tick updates
    void onTick(World world, int x, int y, int z);
}
```

## Block Rotation

Support for rotated blocks:

```java
BlockRotationUtil
├── Rotation calculation
├── Facing direction
└── Flip transformations
```

## Entity Chunks

Entities are stored per-chunk:

```java
EntityChunk entityChunk = column.getEntityChunk();

// Entity iteration
entityChunk.forEachEntity(entity -> {
    // Process entity
});
```

## Environment

Biome/environment data per chunk:

```java
EnvironmentChunk envChunk;
EnvironmentColumn envColumn;
EnvironmentRange envRange;
```

## Lighting

Light data management:

```java
ChunkLightData lightData;
ChunkLightDataBuilder builder;

// Sky light
int skyLight = lightData.getSkyLight(x, y, z);

// Block light
int blockLight = lightData.getBlockLight(x, y, z);
```

## Chunk Flags

Chunk state flags:

```java
public enum ChunkFlag {
    LOADED,
    GENERATED,
    POPULATED,
    // ...
}
```

## Chunk Loading

### Chunk Systems

```java
ChunkSystems
├── LoadBlockSection    // Load section data
├── EnsureBlockSection  // Ensure section exists
├── OnChunkLoad        // Chunk loaded event
├── OnNewChunk         // New chunk generated
├── OnNonTicking       // Chunk became inactive
└── ReplicateChanges   // Sync changes to clients
```

## Data Storage

### DataStoreProvider

```java
DataStoreProvider provider = universe.getDataStoreProvider();

// Types
DiskDataStoreProvider // File-based storage
```

### Player Storage

```java
PlayerStorageProvider storage;

// Per-player data
PlayerStorage playerStorage = storage.getStorage(uuid);
```

## World Commands

Commands for world management:

| Command | Description |
|---------|-------------|
| `/world list` | List all worlds |
| `/world add <name>` | Create new world |
| `/world remove <name>` | Remove world |
| `/world load <name>` | Load world |
| `/world save [name]` | Save world |
| `/world setdefault <name>` | Set default world |
| `/world prune` | Remove empty chunks |

### Block Commands

| Command | Description |
|---------|-------------|
| `/block get <x> <y> <z>` | Get block info |
| `/block set <x> <y> <z> <type>` | Set block |
| `/block getstate` | Get block state |
| `/block setstate` | Set block state |
| `/block select` | Selection tool |
| `/block bulk` | Bulk operations |

## World Generation

Integration with procedural generation:

```java
WorldGenerator generator = world.getGenerator();

// Generate chunk
generator.generate(chunkX, chunkZ);
```

See [World Generation](Builtin-WorldGen) for details.

## Performance

### TPS Commands

```java
/world tps       // Show TPS
/world tps reset // Reset TPS stats
```

### Performance Commands

```java
/world perf       // Show performance
/world perf graph // Performance graph
/world perf reset // Reset stats
```

## Best Practices

1. **Use chunk accessors** - For bulk operations
2. **Cache chunks** - Avoid repeated lookups
3. **Batch block changes** - Reduce sync overhead
4. **Use appropriate palette** - Memory efficiency
5. **Handle chunk boundaries** - Cross-chunk operations
6. **Save periodically** - Prevent data loss
7. **Prune unused chunks** - Disk space management
