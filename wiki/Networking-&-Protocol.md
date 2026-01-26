
The Hytale Server uses a modern networking stack built on Netty with QUIC as the primary protocol and TCP as a fallback. This document covers the networking architecture and protocol system.

## Overview

The networking system is implemented across:
- `com.hypixel.hytale.server.core.io` - Server-side I/O
- `com.hypixel.hytale.protocol` - Protocol definitions

## Architecture

```
ServerManager
├── Transport Layer
│   ├── QUICTransport (UDP-based, primary)
│   └── TCPTransport (fallback)
├── Netty Pipeline
│   ├── PacketDecoder
│   ├── PacketEncoder
│   ├── RateLimitHandler
│   └── PlayerChannelHandler
└── Packet Handlers
    ├── InitialPacketHandler
    ├── HandshakeHandler
    ├── AuthenticationPacketHandler
    ├── SetupPacketHandler
    └── GamePacketHandler
```

## Key Classes

### Server I/O

| Class | Location | Description |
|-------|----------|-------------|
| `ServerManager` | `server.core.io` | Network server management |
| `PacketHandler` | `server.core.io` | Connection packet handling |
| `QUICTransport` | `server.core.io.transport` | QUIC protocol transport |
| `TCPTransport` | `server.core.io.transport` | TCP protocol transport |
| `HytaleChannelInitializer` | `server.core.io.netty` | Netty channel setup |

### Protocol

| Class | Location | Description |
|-------|----------|-------------|
| `PacketDecoder` | `protocol.io.netty` | Packet deserialization |
| `PacketEncoder` | `protocol.io.netty` | Packet serialization |
| `PacketIO` | `protocol.io` | Packet I/O utilities |
| `VarInt` | `protocol.io` | Variable-length integer encoding |
| `ProtocolVersion` | `server.core.io` | Protocol version management |

## Transport Layer

### QUIC Transport (Primary)

QUIC provides:
- Lower latency (0-RTT connection establishment)
- Multiplexed streams
- Built-in encryption
- Better congestion control

```java
QUICTransport
├── QuicChannelInboundHandlerAdapter
└── Netty QUIC codec
```

### TCP Transport (Fallback)

Traditional TCP for compatibility:

```java
TCPTransport
└── Standard Netty TCP channel
```

### Transport Type Enum

```java
public enum TransportType {
    QUIC,
    TCP
}
```

## Connection Flow

```
Client                                Server
   |                                    |
   |-------- Handshake Request -------->|
   |                                    |
   |<------- Handshake Response --------|
   |                                    |
   |-------- Auth Token --------------->|
   |                                    |
   |<------- Auth Response -------------|
   |                                    |
   |-------- Setup Complete ----------->|
   |                                    |
   |<------- World Data ----------------|
   |                                    |
   |<======= Game Packets =============>|
```

## Packet Handlers

### Handler Hierarchy

```
IPacketHandler
└── GenericPacketHandler
    ├── GenericConnectionPacketHandler
    │   └── InitialPacketHandler
    │       └── HandshakeHandler
    ├── AuthenticationPacketHandler
    ├── SetupPacketHandler
    └── GamePacketHandler
        └── InventoryPacketHandler
```

### HandshakeHandler

Manages initial connection:

```java
public class HandshakeHandler extends InitialPacketHandler {
    enum AuthState {
        HANDSHAKE,
        AUTHENTICATION,
        SETUP,
        GAME
    }
}
```

### AuthenticationPacketHandler

Handles player authentication with session service.

### GamePacketHandler

Processes in-game packets for connected players.

## Packet Types

### Asset Editor Packets
Located in `protocol.packets.asseteditor`:
- Asset CRUD operations
- Asset pack management
- Live editing synchronization

### Asset Update Packets
Located in `protocol.packets.assets`:
- `UpdateBlockTypes`
- `UpdateEntityEffects`
- `UpdateEnvironments`
- `UpdateItems`
- `UpdateWeapons`
- And many more...

### Game Packets
Various game state synchronization packets.

## Packet Structure

### VarInt Encoding

Variable-length integers for efficient encoding:

```java
// Write VarInt
VarInt.write(buffer, value);

// Read VarInt
int value = VarInt.read(buffer);
```

### Packet Format

```
┌────────────┬────────────┬─────────────────┐
│ Length     │ Packet ID  │ Payload         │
│ (VarInt)   │ (VarInt)   │ (bytes)         │
└────────────┴────────────┴─────────────────┘
```

## Netty Pipeline

### Channel Initializer

```java
HytaleChannelInitializer
├── PacketDecoder       // Decode incoming packets
├── PacketEncoder       // Encode outgoing packets
├── PacketArrayEncoder  // Batch packet encoding
├── RateLimitHandler    // Rate limiting
├── LatencySimulation   // (Debug) Artificial latency
└── PlayerChannelHandler // Player connection handling
```

### Exception Handling

```java
HytaleChannelInitializer.ExceptionHandler
└── Handles pipeline exceptions
```

## Rate Limiting

Protection against packet flooding:

```java
RateLimitHandler
├── Per-connection limits
├── Configurable thresholds
└── Automatic disconnection
```

Configuration in `HytaleServerConfig`:

```json
{
    "rateLimits": {
        "packetsPerSecond": 1000,
        "bytesPerSecond": 1000000
    }
}
```

## Timeout Configuration

```java
public class TimeoutProfile {
    // Connection timeouts
    // Read/write timeouts
    // Keep-alive settings
}
```

## Packet Statistics

Tracking packet metrics:

```java
PacketStatsRecorder
├── PacketStatsEntry
│   ├── Packet count
│   ├── Byte count
│   └── Size records
└── RecentStats
```

Implementation:

```java
PacketStatsRecorderImpl
└── Per-packet type statistics
```

## Network Serialization

### Serializable Interface

```java
public interface NetworkSerializable {
    void write(ByteBuf buffer);
    void read(ByteBuf buffer);
}
```

### Serializers

```java
NetworkSerializers
├── Primitive serializers
├── Collection serializers
├── Custom type serializers
└── Codec-based serializers
```

## Protocol Version

Version compatibility management:

```java
ProtocolVersion
├── Current version
├── Minimum supported
└── Version negotiation
```

## Packet Adapters

For packet filtering and watching:

```java
PacketAdapters
├── PacketFilter      // Filter outgoing packets
├── PacketWatcher     // Observe packets
├── PlayerPacketFilter
└── PlayerPacketWatcher
```

## Disconnect Reasons

```java
enum DisconnectReason {
    TIMEOUT,
    KICKED,
    BANNED,
    SERVER_SHUTDOWN,
    PROTOCOL_ERROR,
    // ...
}
```

## Ping Information

```java
class PingInfo {
    long latency;
    long lastPingTime;
    // ...
}
```

## Default Configuration

| Setting | Default Value |
|---------|---------------|
| Port | 5520 |
| Protocol | QUIC (primary) |
| Max Players | Configurable |

## Latency Simulation

For testing purposes:

```java
LatencySimulationHandler
├── DelayedRead
├── DelayedWrite
├── DelayedFlush
└── Configurable delay
```

## Best Practices

1. **Use QUIC when possible** - Lower latency
2. **Batch packets** - Reduce overhead
3. **Respect rate limits** - Avoid disconnection
4. **Handle disconnection** - Graceful cleanup
5. **Validate packets** - Security first
6. **Monitor statistics** - Performance tuning

## Security Considerations

- All connections encrypted (QUIC TLS, TCP optional)
- Session authentication required
- Rate limiting prevents DoS
- Packet validation prevents exploits
- Server-authoritative game state

## Debug Commands

```
/bindings - Show network bindings
```
