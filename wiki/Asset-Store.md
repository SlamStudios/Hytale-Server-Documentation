
The Hytale Server uses a sophisticated asset management system for loading, storing, and hot-reloading game assets. This document covers the asset store architecture.

## Overview

The asset system is implemented in:
- `com.hypixel.hytale.assetstore` - Core asset management
- `com.hypixel.hytale.server.core.asset` - Server asset integration

## Architecture

```
AssetStore
├── AssetPack (asset package)
│   ├── AssetMap (key-value storage)
│   │   ├── DefaultAssetMap
│   │   ├── IndexedAssetMap
│   │   └── LookupTableAssetMap
│   └── AssetRegistry (type registry)
├── AssetMonitor (hot-reload)
│   ├── PathWatcherThread
│   └── FileChangeTask
└── Codecs
    ├── AssetCodec
    ├── AssetBuilderCodec
    └── ContainedAssetCodec
```

## Key Classes

### Core

| Class | Location | Description |
|-------|----------|-------------|
| `AssetStore` | `assetstore` | Central asset storage |
| `AssetPack` | `assetstore` | Asset package container |
| `AssetRegistry` | `assetstore` | Asset type registration |
| `AssetMap` | `assetstore` | Key-value asset storage |
| `JsonAsset` | `assetstore` | JSON-based asset |
| `RawAsset` | `assetstore` | Raw binary asset |
| `DecodedAsset` | `assetstore` | Decoded asset data |

### Monitoring

| Class | Description |
|-------|-------------|
| `AssetMonitor` | Watches for asset changes |
| `AssetMonitorHandler` | Handles change events |
| `PathWatcherThread` | File system watcher |
| `FileChangeTask` | Change processing task |

### Codecs

| Class | Description |
|-------|-------------|
| `AssetCodec` | Asset serialization |
| `AssetBuilderCodec` | Builder pattern codec |
| `ContainedAssetCodec` | Nested asset codec |
| `AssetCodecMapCodec` | Map of assets codec |

## AssetStore

Central asset management:

```java
AssetStore store = AssetStore.builder()
    .withPack(assetPack)
    .build();

// Access assets
JsonAsset asset = store.get(AssetType.class, "asset_key");

// Iterate assets
store.forEach(AssetType.class, (key, asset) -> {
    // Process asset
});
```

### Builder Pattern

```java
AssetStore.Builder builder = new AssetStore.Builder();
builder.withPack(pack1);
builder.withPack(pack2);
AssetStore store = builder.build();
```

## AssetPack

Individual asset package:

```java
AssetPack pack;

// Pack info
String name = pack.getName();
boolean immutable = pack.isImmutable();
PluginManifest manifest = pack.getManifest();

// Access assets
JsonAsset asset = pack.get(assetKey);
```

## Asset Maps

### DefaultAssetMap

Standard key-value storage:

```java
DefaultAssetMap<K, V> map;
map.put(key, value);
V value = map.get(key);
```

### IndexedAssetMap

With numeric indexing:

```java
IndexedAssetMap<K, V> map;
map.put(key, value);
V value = map.getByIndex(index);
int index = map.getIndex(key);
```

### LookupTableAssetMap

Fast lookup with pre-computed table:

```java
LookupTableAssetMap<K, V> map;
// Optimized for read-heavy workloads
```

### BlockTypeAssetMap

Specialized for block types:

```java
BlockTypeAssetMap map;
// Efficient block type lookups
```

## Asset Types

### BlockType

```java
BlockType blockType;

// Properties
String name;
boolean solid;
boolean transparent;
float hardness;
// Many more...
```

### ItemType

```java
ItemType itemType;

// Properties
String name;
int stackSize;
float attackDamage;
// Durability, etc.
```

### EntityType

```java
EntityType entityType;

// Properties
String name;
float health;
float speed;
// Behaviors, etc.
```

## JSON Assets

Assets defined in JSON:

```java
public class JsonAsset<K> {
    K key;
    JsonObject data;

    // Inheritance support
    @Nullable String parent;
}
```

### Asset Inheritance

```json
{
    "parent": "base_asset",
    "property": "overridden_value"
}
```

## Asset Loading

### Load Events

```java
LoadAssetEvent event;

// Asset validation
if (event.isShouldShutdown()) {
    List<String> reasons = event.getReasons();
    // Handle validation failure
}
```

### Asset Events

| Event | Description |
|-------|-------------|
| `AssetStoreEvent` | Store operation |
| `RegisterAssetStoreEvent` | Store registered |
| `RemoveAssetStoreEvent` | Store removed |
| `GenerateAssetsEvent` | Assets generated |
| `LoadedAssetsEvent` | Assets loaded |
| `RemovedAssetsEvent` | Assets removed |
| `AssetMonitorEvent` | File change detected |

## Asset Monitoring (Hot-Reload)

### AssetMonitor

Watches for file changes:

```java
AssetMonitor monitor;

// Register handler
monitor.register(path, handler);

// File events
PathEvent event;
EventKind kind = event.getKind(); // CREATE, MODIFY, DELETE
```

### EventKind

```java
public enum EventKind {
    CREATE,
    MODIFY,
    DELETE
}
```

### PathWatcherThread

Background file watching:

```java
PathWatcherThread watcher;
watcher.watch(directory);
watcher.unwatch(directory);
```

## Asset Codecs

### AssetCodec

Base codec for assets:

```java
AssetCodec<T> codec;
T asset = codec.decode(input);
codec.encode(asset, output);
```

### AssetBuilderCodec

Builder pattern support:

```java
AssetBuilderCodec<T> codec;
T.Builder builder = codec.builder();
// Configure builder
T asset = builder.build();
```

### ContainedAssetCodec

For nested assets:

```java
ContainedAssetCodec<T> codec;

// Modes
enum Mode {
    INLINE,
    REFERENCE
}
```

## Asset Validation

### AssetKeyValidator

Validates asset keys:

```java
AssetKeyValidator validator;
boolean valid = validator.isValid(key);
```

### AssetValidationResults

Validation outcome:

```java
AssetValidationResults results;
boolean hasErrors = results.hasErrors();
List<String> errors = results.getErrors();
```

## Asset References

Referencing other assets:

```java
AssetReferences refs;

// Resolve reference
Asset resolved = refs.resolve(reference);
```

## Asset Update Query

Query for asset updates:

```java
AssetUpdateQuery query;

// Rebuild cache
RebuildCache cache = query.getRebuildCache();
```

## Server Asset Types

### Common Assets

```java
CommonAsset asset;
CommonAssetModule module;
CommonAssetRegistry registry;

// Player-specific
PlayerCommonAssets playerAssets;
```

### Asset Packet Generation

```java
AssetPacketGenerator generator;
DefaultAssetPacketGenerator defaultGen;
SimpleAssetPacketGenerator simpleGen;

// Generate update packets
Packet packet = generator.generate(asset);
```

## File Types

Supported asset file types:

```java
HytaleFileTypes
├── JSON (.json)
├── Model (.hym)
├── Animation (.hya)
├── Sound (.ogg)
└── Image (.png)
```

### Sound Validation

```java
SoundFileValidators
└── ChannelValidator (mono/stereo)
```

### OGG Info Cache

```java
OggVorbisInfoCache cache;
OggVorbisInfo info = cache.getInfo(path);
// Duration, channels, sample rate
```

## Blocky Animation Cache

```java
BlockyAnimationCache cache;
BlockyAnimation animation = cache.get(path);
```

## Asset Types Registry

### Registering Types

```java
AssetRegistry registry = plugin.getAssetRegistry();

// Register asset type
registry.register("my_asset", MyAsset.class, MyAsset.CODEC);
```

### Built-in Types

| Type | Description |
|------|-------------|
| `BlockType` | Block definitions |
| `ItemType` | Item definitions |
| `EntityType` | Entity definitions |
| `WeaponType` | Weapon definitions |
| `AmbienceFX` | Ambience effects |
| `AudioCategory` | Audio categories |
| `Environment` | Environment/biome |
| `BlockSoundSet` | Block sounds |
| `BlockParticleSet` | Block particles |
| And many more... | |

## Best Practices

1. **Use asset packs** - Organize related assets
2. **Enable hot-reload** - For development
3. **Validate assets** - Catch errors early
4. **Use inheritance** - Reduce duplication
5. **Index for performance** - Use IndexedAssetMap
6. **Cache decoded assets** - Avoid re-parsing
7. **Handle missing assets** - Graceful fallbacks

## Asset Iterator

For circular dependency detection:

```java
AssetStoreIterator iterator;

// Throws on circular dependency
try {
    iterator.iterate(asset);
} catch (CircularDependencyException e) {
    // Handle cycle
}
```

## Extra Info

Asset metadata:

```java
AssetExtraInfo info;
AssetExtraInfo.Data data = info.getData();
```
