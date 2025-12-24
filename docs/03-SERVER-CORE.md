# Server Core Components

This document provides a deep dive into the core server modules that power the game engine.

---

## Table of Contents

1. [Module Loading System](#module-loading-system)
2. [GameServer](#gameserver)
3. [GameLoop](#gameloop)
4. [World](#world)
5. [Lattice & Chunks](#lattice--chunks)
6. [EventQueue](#eventqueue)
7. [Database](#database)
8. [WorldClock](#worldclock)

---

## Module Loading System

### require.js

The `require.js` file sets up the global module loading system:

```javascript
// Load global configuration
global.CONFIG = require("./config");

// Helper function to load modules from src/
global.requireModule = function() {
  return require(path.join(__dirname, "src", ...arguments));
}

// Helper function to load data files
global.requireData = function() {
  return require(getDataFile(...arguments))
}

// Get full path to a data file
global.getDataFile = function() {
  return path.join(__dirname, "data", CONFIG.SERVER.CLIENT_VERSION, ...arguments);
}

// Load prototype extensions
requireModule("__proto__");

// Load constants from client data
global.CONST = require("./client/data/740/constants.json")
```

### __proto__.js - Prototype Extensions

Adds useful methods to built-in prototypes:

```javascript
// String formatting
String.prototype.format = function() {
  // "Hello %s".format("World") → "Hello World"
}

// Number clamping
Number.prototype.clamp = function(min, max) {
  return Math.max(min, Math.min(max, this));
}

// Random number generation
Number.prototype.random = function(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// Array random element
Array.prototype.random = function() {
  return this[Math.floor(Math.random() * this.length)];
}
```

---

## GameServer

**File:** `src/gameserver.js`

The GameServer is the main container that orchestrates all server components.

### Constructor

```javascript
const GameServer = function() {
  // Handle graceful shutdown
  process.on("SIGINT", this.scheduleShutdown.bind(this, CONFIG.SERVER.MS_SHUTDOWN_SCHEDULE));
  
  // Initialize core components
  this.database = new Database();
  
  this.gameLoop = new GameLoop(
    CONFIG.SERVER.MS_TICK_INTERVAL,  // 50ms
    this.__loop.bind(this)
  );
  
  this.HTTPServer = new HTTPServer(
    CONFIG.SERVER.HOST,
    CONFIG.SERVER.PORT
  );
  
  this.IPCSocket = new IPCSocket();
  
  // State tracking
  this.__serverStatus = null;
  this.__initialized = null;
}
```

### Server Status States

```javascript
GameServer.prototype.STATUS = new Enum(
  "OPEN",      // Server is accepting connections
  "OPENING",   // Server is starting up
  "CLOSING",   // Server is shutting down
  "CLOSED"     // Server has stopped
);
```

### Key Methods

```javascript
// Initialize and start the server
GameServer.prototype.initialize = function() {
  this.__serverStatus = this.STATUS.OPEN;
  this.__initialized = Date.now();
  
  this.database.initialize();   // Load all game data
  this.gameLoop.initialize();   // Start the tick loop
  this.HTTPServer.listen();     // Accept WebSocket connections
}

// Main game loop callback (called every 50ms)
GameServer.prototype.__loop = function() {
  // 1. Handle network I/O for all connected clients
  this.HTTPServer.websocketServer.socketHandler.flushSocketBuffers();
  
  // 2. Advance the world simulation
  this.world.tick();
}

// Schedule a graceful shutdown
GameServer.prototype.scheduleShutdown = function(milliseconds) {
  this.setServerStatus(this.STATUS.CLOSING);
  
  // Notify all players
  this.world.broadcastMessage("The gameserver is closing in X seconds...");
  
  // Shutdown after delay
  setTimeout(this.shutdown.bind(this), milliseconds);
}
```

---

## GameLoop

**File:** `src/gameloop.js`

The GameLoop manages the consistent 50ms tick rate for the game simulation.

### How It Works

```javascript
const GameLoop = function(interval, callback) {
  this.__interval = interval;   // 50ms
  this.__callback = callback;   // GameServer.__loop
  this.__ticks = 0;             // Total tick count
}

GameLoop.prototype.initialize = function() {
  // Start the interval timer
  this.__timer = setInterval(
    this.__tick.bind(this),
    this.__interval
  );
}

GameLoop.prototype.__tick = function() {
  this.__ticks++;
  this.__callback();
}

// Check if current tick is a multiple (for periodic actions)
GameLoop.prototype.isMod = function(modulus) {
  return this.__ticks % modulus === 0;
}
```

### Tick Timing

At 50ms per tick:
- 20 ticks per second
- 1200 ticks per minute
- Step duration calculations use ticks for movement speed

---

## World

**File:** `src/world.js`

The World class contains the entire game map and all entities.

### Constructor

```javascript
const World = function(worldSize) {
  // Chat channels (default, trade, world, etc.)
  this.channelManager = new ChannelManager();
  
  // Spatial grid of chunks
  this.lattice = new Lattice(worldSize);
  
  // Scheduled events (binary heap)
  this.eventQueue = new EventQueue();
  
  // Day/night cycle
  this.clock = new WorldClock();
  
  // Creature management
  this.creatureHandler = new CreatureHandler();
  
  // Combat resolution
  this.combatHandler = new CombatHandler();
  
  // Delegate spatial functions
  this.getTileFromWorldPosition = this.lattice.getTileFromWorldPosition.bind(this.lattice);
  this.getChunkFromWorldPosition = this.lattice.getChunkFromWorldPosition.bind(this.lattice);
  this.findPath = this.lattice.findPath.bind(this.lattice);
}
```

### Tick Method

```javascript
World.prototype.tick = function() {
  // 1. Advance world time
  this.clock.tick();
  
  // 2. Process scheduled events (decay, conditions, etc.)
  this.eventQueue.tick();
  
  // 3. Process all creature actions
  this.creatureHandler.tick();
}
```

### Key Methods

```javascript
// Add an item to the world
World.prototype.addTopThing = function(position, thing) {
  let tile = this.getTileFromWorldPosition(position);
  if(tile !== null) {
    tile.addTopThing(thing);
  }
}

// Create a splash effect (blood, slime, etc.)
World.prototype.addSplash = function(id, position, fluidType) {
  let splash = gameServer.database.createThing(id);
  splash.setFluidType(fluidType);
  this.addThing(position, splash, 0);  // Add at bottom of stack
}

// Send a magic effect to all observers
World.prototype.sendMagicEffect = function(position, type) {
  if(!this.withinBounds(position)) return;
  this.broadcastPosition(position, new EffectMagicPacket(position, type));
}

// Broadcast a message to all players
World.prototype.broadcastMessage = function(message) {
  this.broadcastPacket(new ServerMessagePacket(message));
}
```

---

## Lattice & Chunks

**File:** `src/lattice.js`, `src/chunk.js`

The Lattice is a 3D grid that organizes the world into Chunks for efficient processing.

### Lattice Structure

```javascript
const Lattice = function(worldSize) {
  this.width = worldSize.width;
  this.height = worldSize.height;
  this.depth = worldSize.depth;
  
  // Create chunk grid
  this.__chunks = new Array(this.width);
  for(let x = 0; x < this.width; x++) {
    this.__chunks[x] = new Array(this.height);
    for(let y = 0; y < this.height; y++) {
      this.__chunks[x][y] = new Chunk(x, y);
    }
  }
}

// Convert world position to chunk
Lattice.prototype.getChunkFromWorldPosition = function(position) {
  let chunkX = Math.floor(position.x / CONFIG.WORLD.CHUNK.WIDTH);
  let chunkY = Math.floor(position.y / CONFIG.WORLD.CHUNK.HEIGHT);
  
  if(!this.withinBounds(chunkX, chunkY)) return null;
  
  return this.__chunks[chunkX][chunkY];
}

// Get tile at exact world position
Lattice.prototype.getTileFromWorldPosition = function(position) {
  let chunk = this.getChunkFromWorldPosition(position);
  if(chunk === null) return null;
  
  return chunk.getTile(position);
}
```

### Chunk Structure

```javascript
const Chunk = function(x, y) {
  this.id = Chunk.prototype.id++;  // Unique identifier
  this.position = new Position(
    x * CONFIG.WORLD.CHUNK.WIDTH,
    y * CONFIG.WORLD.CHUNK.HEIGHT,
    0
  );
  
  // 3D array of tiles [x][y][z]
  this.layers = new Array(CONFIG.WORLD.CHUNK.DEPTH);
  
  // Players observing this chunk
  this.__spectators = new Set();
  
  // Active creatures in this chunk
  this.__creatures = new Set();
}

// Broadcast packet to all spectators
Chunk.prototype.broadcast = function(packet) {
  this.__spectators.forEach(player => player.write(packet));
}

// Serialize chunk data for a player
Chunk.prototype.serialize = function(player) {
  player.write(new ChunkPacket(this));
}

// Add a creature to the chunk
Chunk.prototype.addCreature = function(creature) {
  this.__creatures.add(creature);
  this.internalBroadcast(new CreatureStatePacket(creature));
}
```

### Tile Structure

```javascript
const Tile = function(id, position) {
  this.id = id;           // Ground item ID
  this.position = position;
  this.itemStack = null;  // Items on this tile
  this.zone = null;       // Zone information
  this.house = null;      // House ownership
}

// Check if tile blocks movement
Tile.prototype.isBlockSolid = function() {
  return this.hasFlag(FLAG_BLOCK_SOLID);
}

// Check if tile is in protection zone
Tile.prototype.isProtectionZone = function() {
  return this.zone && this.zone.isProtectionZone();
}

// Add item to tile
Tile.prototype.addTopThing = function(thing) {
  if(this.itemStack === null) {
    this.itemStack = new ItemStack();
  }
  this.itemStack.addTopThing(thing);
  
  // Notify observers
  this.broadcast(new ItemAddPacket(this.position, thing, index));
}
```

---

## EventQueue

**File:** `src/eventqueue.js`

The EventQueue is a priority queue (binary heap) for scheduling timed events.

### How It Works

```javascript
const EventQueue = function() {
  // Binary heap for efficient min-time retrieval
  this.__heap = new BinaryHeap();
}

// Add an event to the queue
EventQueue.prototype.addEvent = function(callback, delay, context) {
  let event = new HeapEvent(
    Date.now() + delay,  // When to execute
    callback,
    context
  );
  this.__heap.push(event);
  return event;  // Return for cancellation
}

// Process all due events
EventQueue.prototype.tick = function() {
  let now = Date.now();
  
  // Execute all events that are due
  while(!this.__heap.isEmpty()) {
    let event = this.__heap.peek();
    
    // Not yet time for this event
    if(event.time > now) break;
    
    // Remove and execute
    this.__heap.pop();
    event.callback.call(event.context);
  }
}

// Cancel a scheduled event
EventQueue.prototype.cancelEvent = function(event) {
  event.cancelled = true;
}
```

### Common Event Uses

```javascript
// Item decay (corpse → bones after 5 minutes)
this.decayEvent = world.eventQueue.addEvent(
  this.decay.bind(this),
  300000,  // 5 minutes
  this
);

// Condition tick (poison damage every 2 seconds)
this.conditionEvent = world.eventQueue.addEvent(
  this.applyDamage.bind(this),
  2000,
  this
);

// Spawn respawn (monster respawns after death)
world.eventQueue.addEvent(
  spawn.respawnMonster.bind(spawn),
  spawn.respawnTime,
  spawn
);
```

---

## Database

**File:** `src/database.js`

The Database loads and manages all game data definitions.

### Initialization

```javascript
Database.prototype.initialize = function() {
  // Load item definitions (JSON with properties)
  this.items = this.__loadItemDefinitions("items");
  
  // Load spell callbacks (JavaScript functions)
  this.spells = this.__loadDefinitions("spells");
  
  // Load rune effects
  this.runes = this.__loadDefinitions("runes");
  
  // Load door scripts
  this.doors = this.__loadDefinitions("doors");
  
  // Load house ownership data
  this.houses = this.__loadHouses("houses");
  
  // Load conditions (poison, haste, etc.)
  this.conditions = this.__loadDefinitions("conditions");
  
  // Load action scripts and attach to prototypes
  this.actionLoader.initialize();
  
  // Parse the world map file
  this.worldParser.load(CONFIG.WORLD.WORLD_FILE);
  
  // Attach time-based events to world clock
  this.actionLoader.attachClockEvents("clock");
  
  // Load house item data
  this.loadHouseItems();
  
  // Load monster definitions
  this.monsters = this.__loadDefinitions("monsters");
  
  // Load and spawn NPCs (if enabled)
  if(CONFIG.WORLD.NPCS.ENABLED) {
    this.npcs = this.__loadNPCDefinitions("npcs");
  }
}
```

### Creating Things

```javascript
// Create an item instance from an ID
Database.prototype.createThing = function(id) {
  // Get the prototype (shared properties)
  if(!this.items.hasOwnProperty(id)) return null;
  
  // Create appropriate class based on type
  let thing = this.__createClassFromId(id);
  
  // Set weight for pickupable items
  if(thing.isPickupable()) {
    thing.setWeight(thing.getPrototype().properties.weight);
  }
  
  // Schedule decay if applicable
  if(thing.isDecaying()) {
    thing.scheduleDecay();
  }
  
  return thing;
}

// Determine class type from ID
Database.prototype.__createClassFromId = function(id) {
  let proto = this.getThingPrototype(id);
  
  if(proto.properties === null) {
    return new Item(id);
  }
  
  switch(proto.properties.type) {
    case "corpse": return new Corpse(id, proto.properties.containerSize || 4);
    case "container": return new Container(id, proto.properties.containerSize || 4);
    case "fluidContainer": return new FluidContainer(id);
    case "rune": return new Rune(id);
    case "key": return new Key(id);
    case "door": return new Door(id);
    case "readable": return new Readable(id);
    case "teleport": return new Teleporter(id);
    default: return new Item(id);
  }
}
```

### Retrieving Data

```javascript
// Get monster definition
Database.prototype.getMonster = function(id) {
  return this.monsters.has(id) ? this.monsters.get(id) : null;
}

// Get spell function
Database.prototype.getSpell = function(sid) {
  return this.spells.hasOwnProperty(sid) ? this.spells[sid] : null;
}

// Get item prototype (shared properties)
Database.prototype.getThingPrototype = function(id) {
  return this.items.hasOwnProperty(id) ? this.items[id] : null;
}
```

---

## WorldClock

**File:** `src/world-clock.js`

The WorldClock manages the in-game day/night cycle.

### Configuration

```json
{
  "WORLD": {
    "CLOCK": {
      "SPEED": 6,         // 6x real time
      "START": "08:00"    // Server start time
    }
  }
}
```

### Implementation

```javascript
const WorldClock = function() {
  // Parse starting time
  let [hours, minutes] = CONFIG.WORLD.CLOCK.START.split(":");
  this.__startOffset = (parseInt(hours) * 60 + parseInt(minutes)) * 60 * 1000;
  
  this.__startTime = Date.now();
}

// Called every tick
WorldClock.prototype.tick = function() {
  // Emit time event for listeners
  this.emit("time", this.getTimeString());
}

// Get current in-game time
WorldClock.prototype.getGameTime = function() {
  let elapsed = Date.now() - this.__startTime;
  let gameElapsed = elapsed * CONFIG.WORLD.CLOCK.SPEED;
  return this.__startOffset + gameElapsed;
}

// Get formatted time string
WorldClock.prototype.getTimeString = function() {
  let gameMs = this.getGameTime();
  let totalMinutes = Math.floor(gameMs / 60000) % (24 * 60);
  let hours = Math.floor(totalMinutes / 60);
  let minutes = totalMinutes % 60;
  
  return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
}
```

### Time-Based Events

Scripts can subscribe to time events:

```javascript
// In data/740/clock/definitions/timetile.js
module.exports = function(timeString) {
  let [hours, minutes] = timeString.split(":");
  
  if(hours === "00" && minutes === "00") {
    // Midnight event
    console.log("It's midnight!");
  }
}
```

---

## Summary

The server core consists of:

| Component | Purpose |
|-----------|---------|
| **require.js** | Module loading and globals |
| **GameServer** | Main orchestrator |
| **GameLoop** | 50ms tick timing |
| **World** | Game state container |
| **Lattice/Chunk** | Spatial organization |
| **EventQueue** | Timed events |
| **Database** | Data management |
| **WorldClock** | Day/night cycle |

These components work together to:
1. Load game data on startup
2. Accept player connections
3. Run the game simulation at 50ms intervals
4. Handle events and scheduled actions
5. Manage the game world state

---

*Previous: [Architecture Overview](./02-ARCHITECTURE-OVERVIEW.md) | Next: [Entities →](./04-ENTITIES.md)*

