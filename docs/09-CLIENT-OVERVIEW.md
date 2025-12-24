# Client Architecture Overview

This document explains the HTML5 client architecture and its key components.

---

## Table of Contents

1. [Client Structure](#client-structure)
2. [GameClient](#gameclient)
3. [Rendering System](#rendering-system)
4. [Network Manager](#network-manager)
5. [Interface System](#interface-system)
6. [Input Handling](#input-handling)

---

## Client Structure

```
client/
├── index.html          # Main HTML page
├── css/                # Stylesheets
│   ├── canvas.css
│   ├── equipment.css
│   ├── chatbox.css
│   └── ...
├── png/                # UI images
├── sounds/             # Audio files
├── data/740/           # Client data
│   ├── constants.json  # Shared constants
│   └── *.spr/*.dat     # Sprite/object files
└── src/                # JavaScript modules
    ├── index.js        # Entry point
    ├── gameclient.js   # Main client class
    ├── renderer.js     # Canvas rendering
    ├── network-manager.js
    └── ...
```

---

## GameClient

**File:** `client/src/gameclient.js`

The main client container.

### Structure

```javascript
const GameClient = function() {
  // Sprite/object database
  this.database = new Database();
  
  // UI management
  this.interface = new Interface();
  
  // WebSocket communication
  this.networkManager = new NetworkManager();
  
  // Canvas rendering
  this.renderer = new Renderer();
  
  // Game world state
  this.world = new World();
  
  // Input handlers
  this.keyboard = new Keyboard();
  this.mouse = new Mouse();
  
  // Chat system
  this.channelManager = new ChannelManager();
  
  // Popup dialogs
  this.modalManager = new ModalManager();
  
  // Game loop
  this.gameLoop = new GameLoop();
}
```

### Initialization Flow

```javascript
// 1. Load assets (Tibia.spr, Tibia.dat)
gameClient.database.loadAssets(files);

// 2. Connect to server
gameClient.networkManager.connect(host, token);

// 3. Receive player state
// Handled by packet handlers

// 4. Start game loop
gameClient.gameLoop.start();
```

---

## Rendering System

### Canvas Setup

```javascript
const Canvas = function() {
  // Main game screen (480×352)
  this.screen = document.getElementById("screen");
  this.context = this.screen.getContext("2d");
  
  // Lighting overlay
  this.lightCanvas = new LightCanvas();
  
  // Weather effects
  this.weatherCanvas = new WeatherCanvas();
  
  // Minimap
  this.minimap = new Minimap();
}
```

### Renderer

**File:** `client/src/renderer.js`

```javascript
const Renderer = function() {
  this.canvas = new Canvas();
}

Renderer.prototype.render = function() {
  // 1. Clear canvas
  this.context.clearRect(0, 0, 480, 352);
  
  // 2. Draw ground tiles
  this.renderGround();
  
  // 3. Draw items (bottom to top)
  this.renderItems();
  
  // 4. Draw creatures
  this.renderCreatures();
  
  // 5. Draw effects/animations
  this.renderEffects();
  
  // 6. Draw lighting
  this.lightCanvas.render();
  
  // 7. Draw weather
  this.weatherCanvas.render();
}
```

### Sprite System

**File:** `client/src/sprite.js`

```javascript
const Sprite = function(id) {
  this.id = id;
  this.frameGroups = [];  // Animation frames
}

// Draw sprite to canvas
Sprite.prototype.draw = function(context, x, y, frame) {
  let frameGroup = this.frameGroups[frame % this.frameGroups.length];
  
  context.drawImage(
    frameGroup.canvas,
    0, 0, 32, 32,
    x, y, 32, 32
  );
}
```

---

## Network Manager

**File:** `client/src/network-manager.js`

### Connection

```javascript
const NetworkManager = function() {
  this.socket = null;
  this.connected = false;
}

NetworkManager.prototype.connect = function(host, token) {
  // Connect with token in URL
  this.socket = new WebSocket(host + "?token=" + token);
  
  this.socket.binaryType = "arraybuffer";
  
  this.socket.onopen = this.handleOpen.bind(this);
  this.socket.onclose = this.handleClose.bind(this);
  this.socket.onmessage = this.handleMessage.bind(this);
}
```

### Packet Handling

```javascript
NetworkManager.prototype.handleMessage = function(event) {
  // Decompress if needed
  let data = this.decompress(event.data);
  
  // Parse packets
  let reader = new PacketReader(data);
  
  while(!reader.isEnd()) {
    let opcode = reader.readUInt8();
    gameClient.packetHandler.handlePacket(opcode, reader);
  }
}

// Send packet to server
NetworkManager.prototype.send = function(packet) {
  if(this.connected) {
    this.socket.send(packet.buffer);
  }
}
```

---

## Interface System

**File:** `client/src/interface.js`

### Structure

```javascript
const Interface = function() {
  // Equipment slots
  this.equipment = new Equipment();
  
  // Skill window
  this.skillWindow = new SkillWindow();
  
  // Battle list
  this.battleWindow = new BattleWindow();
  
  // Hotbar (F1-F12 keys)
  this.hotbar = new HotbarManager();
  
  // Status bars (health, mana)
  this.statusBar = new StatusBar();
}
```

### Modals

**File:** `client/src/modal-manager.js`

```javascript
const ModalManager = function() {
  this.modals = new Map();
  
  // Register modals
  this.register("outfit", new OutfitModal());
  this.register("move-item", new MoveItemModal());
  this.register("trade", new TradeModal());
}

ModalManager.prototype.open = function(name, data) {
  let modal = this.modals.get(name);
  modal.show(data);
}

ModalManager.prototype.close = function(name) {
  let modal = this.modals.get(name);
  modal.hide();
}
```

### Windows

Draggable windows for containers, skills, etc.

```javascript
const Window = function(element) {
  this.element = element;
  this.header = element.querySelector(".header");
  this.body = element.querySelector(".body");
  
  // Make draggable
  this.header.addEventListener("mousedown", this.startDrag.bind(this));
}

Window.prototype.startDrag = function(event) {
  // Implement drag behavior
}

Window.prototype.minimize = function() {
  this.body.style.display = "none";
}
```

---

## Input Handling

### Keyboard

**File:** `client/src/keyboard.js`

```javascript
const Keyboard = function() {
  document.addEventListener("keydown", this.handleKeyDown.bind(this));
  document.addEventListener("keyup", this.handleKeyUp.bind(this));
}

Keyboard.prototype.handleKeyDown = function(event) {
  switch(event.keyCode) {
    // Arrow keys - movement
    case 37: gameClient.player.move(CONST.DIRECTION.WEST); break;
    case 38: gameClient.player.move(CONST.DIRECTION.NORTH); break;
    case 39: gameClient.player.move(CONST.DIRECTION.EAST); break;
    case 40: gameClient.player.move(CONST.DIRECTION.SOUTH); break;
    
    // F1-F12 - hotbar
    case 112: gameClient.hotbar.use(0); break;
    // ...
    
    // Enter - chat focus
    case 13: gameClient.interface.focusChat(); break;
  }
}
```

### Mouse

**File:** `client/src/mouse.js`

```javascript
const Mouse = function() {
  let canvas = document.getElementById("screen");
  
  canvas.addEventListener("click", this.handleClick.bind(this));
  canvas.addEventListener("contextmenu", this.handleRightClick.bind(this));
  canvas.addEventListener("mousedown", this.handleMouseDown.bind(this));
  canvas.addEventListener("mouseup", this.handleMouseUp.bind(this));
}

Mouse.prototype.handleClick = function(event) {
  // Convert screen position to world position
  let worldPos = this.screenToWorld(event.clientX, event.clientY);
  
  // Get tile at position
  let tile = gameClient.world.getTile(worldPos);
  
  // Handle click on tile
  if(tile) {
    this.handleTileClick(tile);
  }
}
```

---

## World State

**File:** `client/src/world.js`

```javascript
const World = function() {
  // Loaded chunks
  this.chunks = new Map();
  
  // All known creatures
  this.creatures = new Map();
  
  // Local player reference
  this.player = null;
  
  // Current animations
  this.animations = [];
}

// Get tile at world position
World.prototype.getTile = function(position) {
  let chunk = this.getChunk(position);
  if(!chunk) return null;
  
  return chunk.getTile(position);
}

// Add creature to world
World.prototype.addCreature = function(creature) {
  this.creatures.set(creature.id, creature);
}

// Remove creature from world
World.prototype.removeCreature = function(id) {
  this.creatures.delete(id);
}
```

---

## Game Loop

**File:** `client/src/game-loop.js`

```javascript
const GameLoop = function() {
  this.running = false;
  this.lastFrame = 0;
}

GameLoop.prototype.start = function() {
  this.running = true;
  this.lastFrame = performance.now();
  requestAnimationFrame(this.tick.bind(this));
}

GameLoop.prototype.tick = function(timestamp) {
  if(!this.running) return;
  
  // Calculate delta time
  let delta = timestamp - this.lastFrame;
  this.lastFrame = timestamp;
  
  // Update game state
  gameClient.update(delta);
  
  // Render frame
  gameClient.renderer.render();
  
  // Schedule next frame
  requestAnimationFrame(this.tick.bind(this));
}
```

---

## Summary

| Component | Purpose |
|-----------|---------|
| **GameClient** | Main container |
| **Database** | Sprite/object storage |
| **NetworkManager** | WebSocket handling |
| **Renderer** | Canvas drawing |
| **Interface** | UI management |
| **Keyboard/Mouse** | Input handling |
| **World** | Game state |
| **GameLoop** | Frame timing |

### Key Files

| File | Description |
|------|-------------|
| `index.html` | Main page, loads all scripts |
| `gameclient.js` | Main client class |
| `renderer.js` | Canvas rendering |
| `network-manager.js` | WebSocket client |
| `packet-handler.js` | Incoming packet routing |
| `protocol.js` | Packet creation |

---

*Previous: [Actions & Events](./08-ACTIONS-EVENTS.md) | Next: [Adding Features →](./10-ADDING-FEATURES.md)*

