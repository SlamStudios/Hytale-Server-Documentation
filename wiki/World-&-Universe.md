# World & Universe

The Hytale Server organizes game content into a hierarchical structure with Universe at the top containing multiple Worlds. This document covers world management, chunks, and storage.

## Overview

The world system is implemented in `com.hypixel.hytale.server.core.universe` and provides:

- Multi-world support
- Chunk-based world storage (32×32×32 block sections)
- Block and entity management
- Procedural generation integration
- Data persistence

## Hierarchy

```
Universe (Server)
├── World (Dimension/Realm)
│   ├── ChunkColumn (vertical stack of sections)
│   │   ├── WorldChunk (metadata and state)
│   │   ├── BlockSection[10] (block data per section)
│   │   ├── EntityChunk (entities)
│   │   ├── EnvironmentChunk (biome/environment data)
│   │   └── BlockComponentChunk (block components/states)
│   ├── WorldConfig
│   └── WorldGenerator
└── DataStoreProvider
    └── DiskDataStoreProvider
```

---

## Core Constants

These constants define the chunk system dimensions. You can use them directly or via `ChunkUtil`.

```java
// Chunk/Section dimensions
int SIZE = 32;                    // Chunk width/height/depth
int SIZE_MASK = 31;               // 0x1F - bitmask for local coordinates
int BITS = 5;                     // Bit shift amount (2^5 = 32)

// Derived values
int SIZE_BLOCKS = 32768;          // 32 * 32 * 32 - blocks per section
int SIZE_COLUMNS = 1024;          // 32 * 32 - columns per chunk

// World height
int HEIGHT = 320;                 // Total world height in blocks
int HEIGHT_SECTIONS = 10;         // Number of vertical sections (320 / 32)
int MIN_Y = 0;                    // Minimum Y coordinate
int MAX_Y = 319;                  // Maximum Y coordinate (HEIGHT - 1)
```

---

## Coordinate System

### Converting World to Chunk Coordinates

```java
// Using bit operations directly (how the engine does it internally)
int chunkX = blockX >> 5;         // Divide by 32
int chunkZ = blockZ >> 5;

// Using ChunkUtil
int chunkX = ChunkUtil.chunkCoordinate(blockX);
int chunkZ = ChunkUtil.chunkCoordinate(blockZ);

// For double coordinates (player positions)
int chunkX = ChunkUtil.chunkCoordinate(playerX);  // Floors then shifts
```

### Converting World to Local Coordinates

```java
// Using bit operations directly
int localX = blockX & 31;         // Same as blockX % 32 for positive numbers
int localY = blockY & 31;         // But works correctly for negative numbers
int localZ = blockZ & 31;

// Using ChunkUtil
int localX = ChunkUtil.localCoordinate(blockX);
```

### Converting Local Back to World Coordinates

```java
// Using bit operations directly (from WorldChunk.setBlock)
int worldX = (chunkX << 5) + (localX & 31);
int worldZ = (chunkZ << 5) + (localZ & 31);

// Or equivalently
int worldX = (chunkX << 5) | localX;  // When localX is already 0-31

// Using ChunkUtil
int worldX = ChunkUtil.worldCoordFromLocalCoord(chunkX, localX);
```

### Section Y Index

```java
// Get which section (0-9) contains a Y coordinate
int sectionIndex = blockY >> 5;   // Direct
int sectionIndex = ChunkUtil.indexSection(blockY);  // Via ChunkUtil

// Get the base Y of a section
int baseY = sectionIndex << 5;    // sectionIndex * 32

// Examples:
// Y=0-31   → section 0
// Y=32-63  → section 1
// Y=64-95  → section 2
// Y=288-319 → section 9
```

---

## Block Indexing

Blocks within a section are stored in a flat array of 32,768 elements. The index is calculated from local coordinates.

### Block Index Within Section (0-32767)

```java
// Formula: (y & 31) << 10 | (z & 31) << 5 | (x & 31)

// Using bit operations directly
int index = (localY << 10) | (localZ << 5) | localX;

// Using ChunkUtil
int index = ChunkUtil.indexBlock(localX, localY, localZ);

// Reverse - extract coordinates from index
int x = index & 31;               // ChunkUtil.xFromIndex(index)
int y = (index >> 10) & 31;       // ChunkUtil.yFromIndex(index)
int z = (index >> 5) & 31;        // ChunkUtil.zFromIndex(index)
```

### Column Index Within Chunk (0-1023)

```java
// Formula: (z & 31) << 5 | (x & 31)

// Using bit operations
int columnIndex = (localZ << 5) | localX;

// Using ChunkUtil
int columnIndex = ChunkUtil.indexColumn(localX, localZ);

// Reverse
int x = columnIndex & 31;         // ChunkUtil.xFromColumn(columnIndex)
int z = (columnIndex >> 5) & 31;  // ChunkUtil.zFromColumn(columnIndex)
```

### Chunk Index for Maps (long)

```java
// Pack two ints into a long for use as map key
long chunkIndex = ((long)chunkX << 32) | (chunkZ & 0xFFFFFFFFL);

// Using ChunkUtil
long chunkIndex = ChunkUtil.indexChunk(chunkX, chunkZ);
long chunkIndex = ChunkUtil.indexChunkFromBlock(blockX, blockZ);  // From world coords

// Unpack
int chunkX = (int)(chunkIndex >> 32);   // ChunkUtil.xOfChunkIndex(chunkIndex)
int chunkZ = (int)chunkIndex;           // ChunkUtil.zOfChunkIndex(chunkIndex)
```

---

## ChunkUtil Helper Methods

The `ChunkUtil` class (`com.hypixel.hytale.math.util.ChunkUtil`) provides utility methods for chunk coordinate operations. These methods are simple inline bit operations that the JVM will optimize identically to hand-written code, so use whichever approach fits your code style.

### Coordinate Methods

| Method | Equivalent | Description |
|--------|-----------|-------------|
| `chunkCoordinate(int block)` | `block >> 5` | World to chunk coordinate |
| `chunkCoordinate(double block)` | `floor(block) >> 5` | Double to chunk coordinate |
| `localCoordinate(long v)` | `(int)(v & 31L)` | World to local coordinate |
| `indexSection(int y)` | `y >> 5` | Y to section index |
| `worldCoordFromLocalCoord(chunk, local)` | `(chunk << 5) \| local` | Local to world |
| `minBlock(int chunkCoord)` | `chunkCoord << 5` | First block in chunk |
| `maxBlock(int chunkCoord)` | `(chunkCoord << 5) + 31` | Last block in chunk |

### Indexing Methods

| Method | Description |
|--------|-------------|
| `indexBlock(x, y, z)` | Local coords → section index (0-32767) |
| `indexColumn(x, z)` | Local coords → column index (0-1023) |
| `indexBlockInColumn(x, y, z)` | Coords → full column index (0-327679) |
| `indexChunk(x, z)` | Chunk coords → long key |
| `indexChunkFromBlock(x, z)` | World coords → chunk long key |
| `xFromIndex(index)` / `yFromIndex` / `zFromIndex` | Extract coords from section index |
| `xFromColumn(index)` / `zFromColumn(index)` | Extract coords from column index |
| `xOfChunkIndex(long)` / `zOfChunkIndex(long)` | Extract coords from chunk key |

### Validation Methods

| Method | Description |
|--------|-------------|
| `isWithinLocalChunk(x, z)` | True if x,z both in 0-31 range |
| `isBorderBlock(x, z)` | True if on chunk edge (x/z is 0 or 31) |
| `isBorderBlockGlobal(x, z)` | Same but auto-masks world coords |
| `isSameChunk(x0, z0, x1, z1)` | True if both positions in same chunk |
| `isSameChunkSection(x0,y0,z0, x1,y1,z1)` | True if same 32³ section |
| `isInsideChunk(chunkX, chunkZ, blockX, blockZ)` | True if block is in specified chunk |

---

## Practical Examples

### Example 1: Get Block at World Position

```java
// Simple direct approach
public int getBlockAt(World world, int x, int y, int z) {
    // Bounds check
    if (y < 0 || y >= 320) {
        return 0;  // Air
    }
    
    // Get chunk - can use either approach:
    
    // Option A: Direct bit operations
    long chunkIndex = ((long)(x >> 5) << 32) | ((z >> 5) & 0xFFFFFFFFL);
    
    // Option B: ChunkUtil
    long chunkIndex = ChunkUtil.indexChunkFromBlock(x, z);
    
    WorldChunk chunk = world.getChunk(chunkIndex);
    if (chunk == null) {
        return 0;
    }
    
    return chunk.getBlock(x, y, z);
}
```

### Example 2: Iterate All Blocks in a Section

```java
public void processSection(BlockSection section, int sectionIndex) {
    // Skip empty sections
    if (section.isSolidAir()) {
        return;
    }
    
    int baseY = sectionIndex << 5;  // or: sectionIndex * 32
    
    // Option A: Triple nested loop with coordinate access
    for (int localY = 0; localY < 32; localY++) {
        for (int localZ = 0; localZ < 32; localZ++) {
            for (int localX = 0; localX < 32; localX++) {
                int blockId = section.get(localX, localY, localZ);
                if (blockId != 0) {
                    int worldY = baseY + localY;
                    // process block...
                }
            }
        }
    }
    
    // Option B: Flat index loop (slightly faster, used internally)
    for (int index = 0; index < 32768; index++) {
        int blockId = section.get(index);
        if (blockId != 0) {
            int localX = index & 31;
            int localY = (index >> 10) & 31;
            int localZ = (index >> 5) & 31;
            int worldY = baseY + localY;
            // process block...
        }
    }
}
```

### Example 3: Check if Two Positions are in Same Chunk

```java
public boolean needsCrossChunkHandling(int x1, int z1, int x2, int z2) {
    // Option A: Direct comparison
    return (x1 >> 5) != (x2 >> 5) || (z1 >> 5) != (z2 >> 5);
    
    // Option B: ChunkUtil (clearer intent)
    return !ChunkUtil.isSameChunk(x1, z1, x2, z2);
}
```

### Example 4: Convert Player Position to Chunk Info

```java
public void printPlayerChunkInfo(double playerX, double playerY, double playerZ) {
    // Floor to get block position
    int blockX = (int) Math.floor(playerX);
    int blockY = (int) Math.floor(playerY);
    int blockZ = (int) Math.floor(playerZ);
    
    // Chunk coordinates
    int chunkX = blockX >> 5;
    int chunkZ = blockZ >> 5;
    
    // Section index (clamped to valid range)
    int sectionY = Math.max(0, Math.min(9, blockY >> 5));
    
    // Local position within chunk
    int localX = blockX & 31;
    int localY = blockY & 31;
    int localZ = blockZ & 31;
    
    System.out.printf("Chunk: [%d, %d], Section: %d, Local: [%d, %d, %d]%n",
        chunkX, chunkZ, sectionY, localX, localY, localZ);
}
```

### Example 5: Fill Region Across Chunks (from actual WorldChunk code pattern)

```java
public void fillRegion(World world, int x1, int y1, int z1, int x2, int y2, int z2, 
                       int blockId, BlockType blockType) {
    for (int y = y1; y <= y2; y++) {
        // Skip invalid Y
        if (y < 0 || y >= 320) continue;
        
        for (int z = z1; z <= z2; z++) {
            for (int x = x1; x <= x2; x++) {
                // Get chunk for this position
                long chunkIndex = ChunkUtil.indexChunkFromBlock(x, z);
                WorldChunk chunk = world.getNonTickingChunk(chunkIndex);
                
                if (chunk != null) {
                    chunk.setBlock(x, y, z, blockId, blockType, 0, 0, 0);
                }
            }
        }
    }
}
```

### Example 6: Efficient Same-Chunk Operations (from actual codebase)

This pattern is used in `WorldChunk.setBlock` for handling multi-block structures:

```java
public void handleFillerBlocks(WorldChunk chunk, int worldX, int worldZ, int y,
                               BlockType blockType, int rotation) {
    // Iterate filler block positions
    FillerBlockUtil.forEachFillerBlock(
        blockType.getHitboxBoundingBox().get(rotation),
        (offsetX, offsetY, offsetZ) -> {
            int blockX = worldX + offsetX;
            int blockY = y + offsetY;
            int blockZ = worldZ + offsetZ;
            
            // Check if filler is in same chunk (avoid chunk lookup)
            if (ChunkUtil.isSameChunk(worldX, worldZ, blockX, blockZ)) {
                // Same chunk - use current chunk directly
                chunk.setBlock(blockX, blockY, blockZ, ...);
            } else {
                // Different chunk - need to look it up
                WorldChunk otherChunk = chunk.getWorld()
                    .getNonTickingChunk(ChunkUtil.indexChunkFromBlock(blockX, blockZ));
                otherChunk.setBlock(blockX, blockY, blockZ, ...);
            }
        }
    );
}
```

### Example 7: Block Tick Processing (from BlockSection.forEachTicking)

```java
public void processTicking(BlockSection section, int sectionIndex) {
    if (!section.hasTicking()) {
        return;
    }
    
    // Section's base Y in world coordinates
    int sectionStartY = sectionIndex << 5;
    
    // Iterate ticking blocks using BitSet
    BitSet tickingBlocks = section.getTickingBlocksCopy();
    for (int index = tickingBlocks.nextSetBit(0); 
         index >= 0; 
         index = tickingBlocks.nextSetBit(index + 1)) {
        
        // Extract local coordinates from flat index
        int localX = index & 31;                    // ChunkUtil.xFromIndex(index)
        int localY = (index >> 10) & 31;            // ChunkUtil.yFromIndex(index)
        int localZ = (index >> 5) & 31;             // ChunkUtil.zFromIndex(index)
        
        // Convert to world Y (note the OR operation, same as add when localY < 32)
        int worldY = localY | sectionStartY;
        
        int blockId = section.get(index);
        // Process tick...
    }
}
```

### Example 8: Height Map Scanning

```java
public short findHighestSolidBlock(WorldChunk chunk, int localX, int localZ) {
    BlockChunk blockChunk = chunk.getBlockChunk();
    
    // Scan from top section down
    for (int sectionY = 9; sectionY >= 0; sectionY--) {
        BlockSection section = blockChunk.getSection(sectionY);
        
        // Skip entirely empty sections
        if (section.isSolidAir()) {
            continue;
        }
        
        // Scan from top of section down
        for (int localY = 31; localY >= 0; localY--) {
            int blockId = section.get(localX, localY, localZ);
            if (blockId != 0) {
                // Found solid block
                return (short)((sectionY << 5) + localY);
            }
        }
    }
    
    return -1;  // No solid blocks in column
}
```

### Example 9: Border Block Detection for Neighbor Updates

```java
public Set<Long> getAffectedChunks(int worldX, int worldZ, int radius) {
    Set<Long> chunks = new HashSet<>();
    
    // Always include center chunk
    int centerChunkX = worldX >> 5;
    int centerChunkZ = worldZ >> 5;
    chunks.add(ChunkUtil.indexChunk(centerChunkX, centerChunkZ));
    
    // Check if radius extends past chunk borders
    int localX = worldX & 31;
    int localZ = worldZ & 31;
    
    // Near -X edge?
    if (localX < radius) {
        chunks.add(ChunkUtil.indexChunk(centerChunkX - 1, centerChunkZ));
    }
    // Near +X edge?
    if (localX > 31 - radius) {
        chunks.add(ChunkUtil.indexChunk(centerChunkX + 1, centerChunkZ));
    }
    // Near -Z edge?
    if (localZ < radius) {
        chunks.add(ChunkUtil.indexChunk(centerChunkX, centerChunkZ - 1));
    }
    // Near +Z edge?
    if (localZ > 31 - radius) {
        chunks.add(ChunkUtil.indexChunk(centerChunkX, centerChunkZ + 1));
    }
    
    return chunks;
}
```

### Example 10: Block Update Propagation (from ChunkAccessor)

```java
// From actual ChunkAccessor.performBlockUpdate implementation
public boolean performBlockUpdate(int x, int y, int z, boolean allowPartialLoad) {
    boolean success = true;
    
    // Update all 26 neighbors + center (3×3×3 cube)
    for (int dx = -1; dx <= 1; dx++) {
        int wx = x + dx;
        for (int dz = -1; dz <= 1; dz++) {
            int wz = z + dz;
            
            // Get chunk for this XZ (may be different from center)
            long chunkIndex = ChunkUtil.indexChunkFromBlock(wx, wz);
            WorldChunk chunk = allowPartialLoad 
                ? getNonTickingChunk(chunkIndex) 
                : getChunkIfInMemory(chunkIndex);
            
            if (chunk == null) {
                success = false;
                continue;
            }
            
            for (int dy = -1; dy <= 1; dy++) {
                int wy = y + dy;
                chunk.setTicking(wx, wy, wz, true);
            }
        }
    }
    
    return success;
}
```

---

## WorldChunk

The primary interface for chunk operations.

```java
WorldChunk chunk = world.getChunk(chunkIndex);

// Position
int chunkX = chunk.getX();
int chunkZ = chunk.getZ();
long index = chunk.getIndex();

// Block access (coordinates can be world or local - internally masked)
int blockId = chunk.getBlock(x, y, z);
chunk.setBlock(x, y, z, blockId, blockType, rotation, filler, settings);
chunk.breakBlock(x, y, z, settings);

// Height map
short height = chunk.getHeight(localX, localZ);

// Block state
BlockState state = chunk.getState(x, y, z);
chunk.setState(x, y, z, state, notify);

// Ticking
chunk.setTicking(x, y, z, true);
boolean ticking = chunk.isTicking(x, y, z);

// Flags
chunk.is(ChunkFlag.TICKING);
chunk.setFlag(ChunkFlag.LOADED, true);
```

### SetBlock Settings Flags

```java
int SKIP_STATE_INIT = 0x01;       // Don't initialize block state
int SKIP_STATE_UPDATE = 0x02;     // Don't update existing state  
int SKIP_PARTICLES = 0x04;        // No break/build particles
int SKIP_FILLER_CLEANUP = 0x08;   // Don't remove multi-block fillers
int SKIP_FILLER_SET = 0x10;       // Don't set filler blocks
int PHYSICS_BREAK = 0x20;         // Use physics particle type
int FORCE_DIRTY = 0x40;           // Force chunk dirty state
int PERFORM_BLOCK_UPDATE = 0x100; // Trigger block updates
int SKIP_HEIGHT_UPDATE = 0x200;   // Don't recalculate heightmap
```

---

## BlockSection

32×32×32 block storage with palette compression.

```java
// Get section for a Y coordinate
int sectionIndex = y >> 5;
BlockSection section = blockChunk.getSection(sectionIndex);

// Block access - coordinate version
int blockId = section.get(localX, localY, localZ);
section.set(localX, localY, localZ, blockId, rotation, filler);

// Block access - index version (faster for iteration)
int blockId = section.get(index);
section.set(index, blockId, rotation, filler);

// Queries
boolean empty = section.isSolidAir();
boolean hasBlock = section.contains(blockId);
int count = section.count();          // Total non-air
int count = section.count(blockId);   // Specific type
IntSet types = section.values();      // All block IDs present

// Ticking
section.setTicking(index, true);
boolean ticking = section.isTicking(index);
int tickCount = section.getTickingBlocksCount();
```

---

## Palette System

Efficient block storage using dynamic palettes:

| Type | Bits/Block | Max Types | Storage | Auto-Demote |
|------|-----------|-----------|---------|-------------|
| Empty | 0 | 1 | 0 bytes | N/A |
| HalfByte | 4 | 16 | 16 KB | ≤1 type |
| Byte | 8 | 256 | 32 KB | ≤14 types |
| Short | 16 | 65,536 | 64 KB | ≤251 types |

The palette automatically promotes/demotes as block variety changes.

---

## Quick Reference

| Property | Value | Bit Operation |
|----------|-------|---------------|
| Chunk Size | 32×32×32 | - |
| World Height | 320 (0-319) | - |
| Sections | 10 | - |
| Blocks per Section | 32,768 | - |
| World → Chunk | `x >> 5` | Divide by 32 |
| World → Local | `x & 31` | Modulo 32 |
| Local → World | `(chunk << 5) \| local` | - |
| Section Index | `y >> 5` | Which section |
| Block Index | `(y&31)<<10 \| (z&31)<<5 \| x&31` | Flat array index |

---

## Best Practices

1. **Choose the right approach**: Use direct bit operations in tight loops for performance; use ChunkUtil for clarity in higher-level code.

2. **Cache chunk references**: When processing multiple blocks in the same area, store the WorldChunk reference instead of looking it up each time.

3. **Use `isSameChunk()` checks**: Before accessing blocks that might cross chunk boundaries, check if you can avoid the chunk lookup.

4. **Check Y bounds**: Always verify `y >= 0 && y < 320` before block operations to avoid silent failures.

5. **Use section queries**: Call `section.contains(blockId)` before searching to skip sections that don't have your target block.

6. **Prefer index-based iteration**: For bulk operations, iterate flat indices (0-32767) and extract coordinates, rather than triple-nested loops.

7. **Handle chunk borders**: Operations near edges (local coord 0 or 31) may need to access neighbor chunks.
