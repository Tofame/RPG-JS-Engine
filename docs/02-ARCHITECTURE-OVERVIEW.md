# Architecture Overview

This document explains the high-level architecture of the RPG-JS-Engine, how components interact, and the flow of data through the system.

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BROWSER (Client)                                │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                          GameClient                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │   │
│  │  │  Renderer   │  │   World     │  │  Interface  │  │  Network   │  │   │
│  │  │  (Canvas)   │  │ (Chunks,    │  │  (Modals,   │  │  Manager   │  │   │
│  │  │             │  │  Creatures) │  │   Menus)    │  │ (WebSocket)│  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ WebSocket (ws://127.0.0.1:2222)
                                    │ Binary Packets
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            NODE.JS (Game Server)                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                          GameServer                                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │   │
│  │  │  HTTPServer │  │   World     │  │  Database   │  │  GameLoop  │  │   │
│  │  │ (WebSocket) │  │ (Lattice,   │  │ (Items,     │  │  (50ms     │  │   │
│  │  │             │  │  Creatures) │  │  Monsters)  │  │   ticks)   │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ HTTP (token validation)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           NODE.JS (Login Server)                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         LoginServer                                   │   │
│  │  ┌─────────────────────────┐  ┌─────────────────────────────────┐   │   │
│  │  │     HTTP Server         │  │      AccountDatabase            │   │   │
│  │  │  (Create Account,       │  │      (SQLite3)                  │   │   │
│  │  │   Get Token)            │  │                                 │   │   │
│  │  └─────────────────────────┘  └─────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Authentication Flow

```
┌────────┐         ┌─────────────┐         ┌────────────┐         ┌────────────┐
│ Client │         │Login Server │         │ Game Server│         │   World    │
└───┬────┘         └──────┬──────┘         └─────┬──────┘         └─────┬──────┘
    │                     │                      │                      │
    │ 1. Login Request    │                      │                      │
    │ (account, password) │                      │                      │
    │────────────────────>│                      │                      │
    │                     │                      │                      │
    │                     │ 2. Validate          │                      │
    │                     │ (bcrypt compare)     │                      │
    │                     │                      │                      │
    │ 3. Return HMAC Token│                      │                      │
    │<────────────────────│                      │                      │
    │                     │                      │                      │
    │ 4. WebSocket Connect│with Token            │                      │
    │────────────────────────────────────────────>│                      │
    │                     │                      │                      │
    │                     │                      │ 5. Verify HMAC       │
    │                     │                      │ Signature            │
    │                     │                      │                      │
    │                     │                      │ 6. Load Character    │
    │                     │                      │────────────────────>│
    │                     │                      │                      │
    │ 7. Game State Packets│                     │                      │
    │<───────────────────────────────────────────│                      │
    │                     │                      │                      │
```

### HMAC Token Structure

```javascript
{
  "name": "111111",           // Account identifier
  "expire": 1703452800000,    // Token expiration timestamp
  "hmac": "abc123..."         // SHA256-HMAC signature
}
```

The token is base64 encoded and validated by the game server using the shared secret.

---

## Server Component Architecture

### GameServer (Main Container)

```
GameServer
├── database          # Data management (items, monsters, NPCs)
├── gameLoop          # 50ms tick timer
├── HTTPServer        # WebSocket server wrapper
│   └── websocketServer
│       └── socketHandler  # Connected client management
└── world             # Game world instance
```

### World (Game World)

```
World
├── lattice           # Spatial grid of chunks
│   └── chunks[][]    # 2D array of Chunk objects
│       └── tiles[][] # Tiles within each chunk
├── eventQueue        # Scheduled game events (binary heap)
├── clock             # Day/night cycle timer
├── channelManager    # Chat channels
├── creatureHandler   # All creatures (players, monsters, NPCs)
└── combatHandler     # Combat resolution
```

### Entity Hierarchy

```
EventEmitter
└── Creature (Base Class)
    ├── Player
    │   ├── socketHandler      # Network communication
    │   ├── containerManager   # Inventory & equipment
    │   ├── movementHandler    # Walking logic
    │   ├── actionHandler      # Use/look/attack actions
    │   ├── combatLock         # Combat state
    │   ├── spellbook          # Learned spells
    │   └── skills             # Character statistics
    │
    ├── Monster
    │   ├── behaviourHandler   # AI (chase, flee, wander)
    │   ├── damageMap          # Track damage sources
    │   └── lootHandler        # Drop items on death
    │
    └── NPC
        ├── conversationHandler # Dialog system
        ├── behaviourHandler    # Wandering
        └── cutsceneHandler     # Scripted actions
```

---

## Data Flow Architecture

### Server Tick Cycle (every 50ms)

```
gameServer.__loop()
    │
    ├─── socketHandler.flushSocketBuffers()
    │    │
    │    └─── For each connected player:
    │         ├─── Read incoming packets
    │         └─── Write outgoing packets
    │
    └─── world.tick()
         │
         ├─── clock.tick()
         │    └─── Emit "time" events for day/night
         │
         ├─── eventQueue.tick()
         │    └─── Execute scheduled events (timers, decays)
         │
         └─── creatureHandler.tick()
              │
              └─── For each active creature:
                   └─── creature.think()
                        ├─── Player: Handle queued actions
                        ├─── Monster: AI decision making
                        └─── NPC: Wandering, conversations
```

### Packet Flow (Client → Server)

```
Client Input (keyboard/mouse)
    │
    ▼
PacketWriter.write() ─────────> Binary Buffer
    │
    ▼
WebSocket.send()
    │
    ▼
gameServer receives data
    │
    ▼
PacketReader.read() ─────────> Parsed Packet Object
    │
    ▼
PacketHandler.handlePacket()
    │
    ├─── MOVEMENT: player.movementHandler.handleMove()
    ├─── USE_ITEM: player.useHandler.handleUse()
    ├─── ATTACK: player.actionHandler.handleAttack()
    └─── CHAT: channelManager.handleMessage()
```

### Packet Flow (Server → Client)

```
Game Event (movement, combat, etc.)
    │
    ▼
new XxxPacket() ─────────> PacketWriter subclass
    │
    ▼
chunk.broadcast(packet) ──> Send to all nearby players
    │
    ▼
player.write(packet)
    │
    ▼
socket.send(packet.buffer)
    │
    ▼
Client receives binary data
    │
    ▼
PacketReader.parse()
    │
    ▼
PacketHandler dispatches to handler
    │
    ▼
Update game state & re-render
```

---

## Spatial Organization

### Chunk-Based World Structure

```
World Lattice
┌─────────────────────────────────────────────────┐
│  ┌─────────┬─────────┬─────────┬─────────┐     │
│  │ Chunk   │ Chunk   │ Chunk   │ Chunk   │     │
│  │ (0,0)   │ (1,0)   │ (2,0)   │ (3,0)   │     │
│  ├─────────┼─────────┼─────────┼─────────┤     │
│  │ Chunk   │ Chunk   │ Chunk   │ Chunk   │     │
│  │ (0,1)   │ (1,1)   │ (2,1)   │ (3,1)   │     │
│  ├─────────┼─────────┼─────────┼─────────┤     │
│  │ Chunk   │ Chunk   │ Chunk   │ Chunk   │     │
│  │ (0,2)   │ (1,2)   │ (2,2)   │ (3,2)   │     │
│  └─────────┴─────────┴─────────┴─────────┘     │
│                                                 │
│  Each chunk contains tiles across multiple      │
│  floors (z-levels) and tracks which             │
│  players are observing it.                      │
└─────────────────────────────────────────────────┘
```

### Chunk Details

```javascript
// Default chunk dimensions (from config)
CONFIG.WORLD.CHUNK = {
  WIDTH: 9,     // Tiles wide
  HEIGHT: 7,    // Tiles tall  
  DEPTH: 8      // Number of floors (z-levels)
}
```

Each chunk:
- Contains a 3D array of tiles (x, y, z)
- Tracks spectators (players who can see this chunk)
- Handles broadcasting packets to all spectators

---

## Event System Architecture

### Event Queue (Priority Queue)

Events are scheduled using a binary heap for efficient time-ordered execution:

```javascript
// Schedule an event for 5 seconds from now
world.eventQueue.addEvent(callback, 5000, context);

// Events are processed every tick
eventQueue.tick() {
  while (nextEvent.time <= currentTime) {
    nextEvent.callback();
  }
}
```

### Event Types

| Event Type | Description | Example |
|------------|-------------|---------|
| Decay | Item transformation over time | Corpse → Bones |
| Condition | Status effect ticks | Poison damage |
| Regeneration | Health/mana recovery | Every 30 seconds |
| Spawn | Monster respawn timer | After death |
| Clock | Time-based world events | Day/night cycle |

### EventEmitter Pattern

Many classes extend EventEmitter for loose coupling:

```javascript
// Creature emits events
creature.emit("death");
creature.emit("move", oldPosition, newPosition);

// Other systems listen
creature.on("death", function() {
  // Handle death (drop loot, give exp, etc.)
});
```

---

## Database Architecture

### Data Loading Flow

```
Database.initialize()
    │
    ├─── Load item definitions (items/definitions.json)
    │    └─── Create ThingPrototype for each item
    │
    ├─── Load spell definitions (spells/definitions.json)
    │
    ├─── Load monster definitions (monsters/definitions.json)
    │
    ├─── Load NPC definitions (npcs/definitions.json)
    │    └─── Create and spawn NPC instances
    │
    ├─── Load action scripts (actions/definitions.json)
    │    └─── Attach event listeners to prototypes
    │
    └─── Parse world file (world/Tibia74.otbm)
         └─── Create chunks and tiles
```

### Item Creation Flow

```javascript
// Creating a new item
let item = gameServer.database.createThing(itemId);

// Internal flow:
Database.createThing(id)
    │
    ├─── Get prototype: this.getThingPrototype(id)
    │
    ├─── Determine class type from prototype properties
    │    ├─── Container → new Container(id, size)
    │    ├─── FluidContainer → new FluidContainer(id)
    │    ├─── Door → new Door(id)
    │    └─── Default → new Item(id)
    │
    └─── Initialize (set weight, schedule decay, etc.)
```

---

## Client Architecture Overview

### Client Components

```
GameClient
├── database          # Cached sprite/data files
├── interface         # UI management
├── networkManager    # WebSocket client
├── renderer          # Canvas drawing
├── world             # Game state
│   ├── chunks        # Loaded map chunks
│   ├── creatures     # All visible creatures
│   └── player        # Local player reference
├── keyboard          # Input handling
├── mouse             # Mouse events
├── channelManager    # Chat system
└── modalManager      # Popup dialogs
```

### Rendering Pipeline

```
GameLoop.tick() (requestAnimationFrame)
    │
    ├─── Update animations
    │    └─── creature.animate(), item.animate()
    │
    ├─── Process network packets
    │    └─── networkManager.processPackets()
    │
    └─── Render frame
         │
         ├─── Clear canvas
         ├─── Draw ground tiles
         ├─── Draw items (bottom to top)
         ├─── Draw creatures (sorted by y-position)
         ├─── Draw animations (magic effects)
         ├─── Draw lighting overlay
         └─── Draw UI elements
```

---

## Summary

The architecture follows these key principles:

1. **Separation of Concerns** - Server (logic), Client (rendering), Data (configuration)
2. **Event-Driven** - Loose coupling through EventEmitter pattern
3. **Tick-Based** - Predictable 50ms game loop for synchronization
4. **Chunk-Based Spatial** - Efficient visibility and broadcasting
5. **Binary Protocol** - Compact packet format for network efficiency
6. **Prototype Pattern** - Shared properties for items and creatures

---

*Previous: [Getting Started](./01-GETTING-STARTED.md) | Next: [Server Core →](./03-SERVER-CORE.md)*

