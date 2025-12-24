# Networking and Protocol

This document explains the network architecture, packet system, and client-server communication.

---

## Table of Contents

1. [Network Architecture](#network-architecture)
2. [WebSocket Server](#websocket-server)
3. [Packet System](#packet-system)
4. [Protocol Packets](#protocol-packets)
5. [Packet Handler](#packet-handler)

---

## Network Architecture

```
┌────────────────┐     WebSocket      ┌─────────────────┐
│  HTML5 Client  │ ◄───────────────► │  Game Server    │
│                │   Binary Packets   │                 │
│  NetworkManager│                    │  HTTPServer     │
│  PacketWriter  │                    │  WebSocketServer│
│  PacketReader  │                    │  SocketHandler  │
└────────────────┘                    └─────────────────┘
```

### Connection Flow

1. Client requests token from Login Server (HTTP)
2. Client connects to Game Server (WebSocket)
3. Client sends token in upgrade request
4. Server validates HMAC signature
5. Connection established, player loaded

---

## WebSocket Server

**File:** `src/websocket-server.js`

### HTTPServer Wrapper

```javascript
const HTTPServer = function(host, port) {
  // Create HTTP server
  this.server = http.createServer();
  
  // Attach WebSocket server
  this.websocketServer = new WebSocketServer(this.server);
}
```

### WebSocket Server

```javascript
const WebSocketServer = function(server) {
  // Create ws server
  this.wss = new WebSocket.Server({ noServer: true });
  
  // Handle upgrades
  server.on("upgrade", this.__handleUpgrade.bind(this));
  
  // Socket handler for all connections
  this.socketHandler = new SocketHandler();
}

// Handle connection upgrade
WebSocketServer.prototype.__handleUpgrade = function(request, socket, head) {
  // Parse token from URL
  let token = this.__parseToken(request.url);
  
  // Verify HMAC signature
  if(!this.__verifyToken(token)) {
    socket.destroy();
    return;
  }
  
  // Upgrade to WebSocket
  this.wss.handleUpgrade(request, socket, head, (ws) => {
    this.__handleConnection(ws, token);
  });
}
```

---

## Packet System

### Packet Structure

All packets follow this structure:

```
┌──────────┬────────┬───────────────────┐
│  Opcode  │ Length │      Payload      │
│  1 byte  │ 2 bytes│   Variable bytes  │
└──────────┴────────┴───────────────────┘
```

### PacketWriter (Server → Client)

**File:** `src/packet-writer.js`

```javascript
const PacketWriter = function(opcode, payloadSize) {
  // Create buffer for packet
  this.__buffer = Buffer.alloc(3 + payloadSize);
  this.__offset = 0;
  
  // Write header
  this.writeUInt8(opcode);
  this.writeUInt16(payloadSize);
}

// Writing methods
PacketWriter.prototype.writeUInt8 = function(value) {
  this.__buffer.writeUInt8(value, this.__offset++);
}

PacketWriter.prototype.writeUInt16 = function(value) {
  this.__buffer.writeUInt16LE(value, this.__offset);
  this.__offset += 2;
}

PacketWriter.prototype.writeUInt32 = function(value) {
  this.__buffer.writeUInt32LE(value, this.__offset);
  this.__offset += 4;
}

PacketWriter.prototype.writePosition = function(position) {
  this.writeUInt16(position.x);
  this.writeUInt16(position.y);
  this.writeUInt8(position.z);
}

PacketWriter.prototype.writeString = function(string) {
  let encoded = this.encodeString(string);
  this.writeBuffer(encoded);
}
```

### PacketReader (Client → Server)

**File:** `src/packet-reader.js`

```javascript
const PacketReader = function(buffer) {
  this.__buffer = buffer;
  this.__offset = 0;
}

// Reading methods
PacketReader.prototype.readUInt8 = function() {
  return this.__buffer.readUInt8(this.__offset++);
}

PacketReader.prototype.readUInt16 = function() {
  let value = this.__buffer.readUInt16LE(this.__offset);
  this.__offset += 2;
  return value;
}

PacketReader.prototype.readPosition = function() {
  return new Position(
    this.readUInt16(),
    this.readUInt16(),
    this.readUInt8()
  );
}

PacketReader.prototype.readString = function() {
  let length = this.readUInt16();
  let string = this.__buffer.toString("utf8", this.__offset, this.__offset + length);
  this.__offset += length;
  return string;
}
```

---

## Protocol Packets

**File:** `src/protocol.js`

All packet types are defined here. Examples:

### Creature Packets

```javascript
// Creature state (full info)
const CreatureStatePacket = function(creature) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.CREATURE_STATE, size);
  
  this.writeUInt32(creature.getId());
  this.writePosition(creature.getPosition());
  this.writeOutfit(creature.getOutfit());
  this.writeUInt32(creature.getProperty(CONST.PROPERTIES.HEALTH));
  // ... more fields
}

// Creature movement
const CreatureMovePacket = function(guid, position, duration) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.CREATURE_MOVE, 12);
  
  this.writeUInt32(guid);
  this.writePosition(position);
  this.writeUInt16(duration);
}
```

### Item Packets

```javascript
// Item added
const ItemAddPacket = function(position, thing, index) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.ITEM_ADD, 10);
  
  this.writeClientId(thing.id);
  this.writeUInt8(thing.count);
  this.writePosition(position);
  this.writeUInt8(index);
}

// Item removed
const ItemRemovePacket = function(position, index, count) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.ITEM_REMOVE, 8);
  
  this.writePosition(position);
  this.writeUInt8(index);
  this.writeUInt8(count);
}
```

### Container Packets

```javascript
// Open container
const ContainerOpenPacket = function(cid, name, container) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.CONTAINER_OPEN, size);
  
  this.writeUInt32(container.guid);
  this.writeClientId(cid);
  this.writeString(name);
  this.writeUInt8(container.size);
  container.getSlots().forEach(this.writeItem, this);
}
```

### Effect Packets

```javascript
// Magic effect at position
const EffectMagicPacket = function(position, type) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.MAGIC_EFFECT, 7);
  
  this.writePosition(position);
  this.writeUInt8(type);
}

// Distance effect (projectile)
const EffectDistancePacket = function(from, to, type) {
  PacketWriter.call(this, CONST.PROTOCOL.SERVER.DISTANCE_EFFECT, 13);
  
  this.writePosition(from);
  this.writePosition(to);
  this.writeUInt8(type);
}
```

---

## Packet Handler

**File:** `src/packet-handler.js`

Processes incoming packets from clients.

### Structure

```javascript
const PacketHandler = function() {
  // Map opcode to handler function
  this.__handlers = new Map();
  this.__registerHandlers();
}

PacketHandler.prototype.__registerHandlers = function() {
  this.__handlers.set(CONST.PROTOCOL.CLIENT.MOVE, this.handleMove);
  this.__handlers.set(CONST.PROTOCOL.CLIENT.USE, this.handleUse);
  this.__handlers.set(CONST.PROTOCOL.CLIENT.ATTACK, this.handleAttack);
  this.__handlers.set(CONST.PROTOCOL.CLIENT.SAY, this.handleSay);
  // ... more handlers
}

PacketHandler.prototype.handlePacket = function(player, packet) {
  let opcode = packet.readUInt8();
  let handler = this.__handlers.get(opcode);
  
  if(handler) {
    handler.call(this, player, packet);
  }
}
```

### Handler Examples

```javascript
// Movement packet
PacketHandler.prototype.handleMove = function(player, packet) {
  let direction = packet.readUInt8();
  player.movementHandler.handleMove(direction);
}

// Use item packet
PacketHandler.prototype.handleUse = function(player, packet) {
  let position = packet.readPosition();
  let index = packet.readUInt8();
  player.useHandler.handleUse(position, index);
}

// Say packet
PacketHandler.prototype.handleSay = function(player, packet) {
  let channelId = packet.readUInt32();
  let message = packet.readString();
  player.channelManager.handleMessage(channelId, message);
}
```

---

## Socket Handler

**File:** `src/websocket-server-socket-handler.js`

Manages all connected sockets.

### Buffer Flushing

```javascript
// Called every tick (50ms)
SocketHandler.prototype.flushSocketBuffers = function() {
  this.__sockets.forEach(function(socket) {
    // Read incoming packets
    socket.flushReadBuffer();
    
    // Write outgoing packets
    socket.flushWriteBuffer();
  });
}
```

### Socket Write

```javascript
// Queue packet for sending
socket.write = function(packet) {
  this.__writeBuffer.push(packet);
}

// Flush write buffer
socket.flushWriteBuffer = function() {
  if(this.__writeBuffer.length === 0) return;
  
  // Combine all packets
  let combined = Buffer.concat(this.__writeBuffer.map(p => p.__buffer));
  
  // Optional: compress if over threshold
  if(CONFIG.SERVER.COMPRESSION.ENABLED && combined.length > CONFIG.SERVER.COMPRESSION.THRESHOLD) {
    combined = zlib.deflateSync(combined, { level: CONFIG.SERVER.COMPRESSION.LEVEL });
  }
  
  // Send
  this.__websocket.send(combined);
  
  // Clear buffer
  this.__writeBuffer = [];
}
```

---

## Summary

| Component | Purpose |
|-----------|---------|
| **HTTPServer** | HTTP server wrapper |
| **WebSocketServer** | WebSocket handling |
| **SocketHandler** | Connection management |
| **PacketWriter** | Create outgoing packets |
| **PacketReader** | Parse incoming packets |
| **PacketHandler** | Route packets to handlers |
| **protocol.js** | All packet class definitions |

### Data Types

| Method | Size | Range |
|--------|------|-------|
| `writeUInt8` | 1 byte | 0-255 |
| `writeUInt16` | 2 bytes | 0-65535 |
| `writeUInt32` | 4 bytes | 0-4294967295 |
| `writePosition` | 5 bytes | x, y, z |
| `writeString` | 2 + n bytes | length prefix |

---

*Previous: [Items & Containers](./05-ITEMS-CONTAINERS.md) | Next: [Data Definitions →](./07-DATA-DEFINITIONS.md)*

