# RPG-JS-Engine Documentation

## Complete Developer Guide

Welcome to the comprehensive documentation for the **Forby HTML5 Open Tibia Server** - a full-stack JavaScript/NodeJS implementation of a classic MMORPG game engine with both server and client components.

---

## 📚 Documentation Index

| Document | Description |
|----------|-------------|
| [01-GETTING-STARTED.md](./01-GETTING-STARTED.md) | Installation, setup, and running the server |
| [02-ARCHITECTURE-OVERVIEW.md](./02-ARCHITECTURE-OVERVIEW.md) | High-level system architecture |
| [03-SERVER-CORE.md](./03-SERVER-CORE.md) | GameServer, World, GameLoop, Database |
| [04-ENTITIES.md](./04-ENTITIES.md) | Creature, Player, Monster, NPC systems |
| [05-ITEMS-CONTAINERS.md](./05-ITEMS-CONTAINERS.md) | Item system, containers, equipment |
| [06-NETWORKING.md](./06-NETWORKING.md) | Protocol, packets, WebSocket communication |
| [07-DATA-DEFINITIONS.md](./07-DATA-DEFINITIONS.md) | How to define monsters, NPCs, items |
| [08-ACTIONS-EVENTS.md](./08-ACTIONS-EVENTS.md) | Action system, scripting, events |
| [09-CLIENT-OVERVIEW.md](./09-CLIENT-OVERVIEW.md) | Client-side architecture |
| [10-ADDING-FEATURES.md](./10-ADDING-FEATURES.md) | How to add new game content |
| [11-FILE-REFERENCE.md](./11-FILE-REFERENCE.md) | Complete file-by-file reference |

---

## 🎮 What Is This Project?

This project is a complete **MMORPG game engine** consisting of:

1. **Game Server** (`engine.js`) - NodeJS WebSocket server handling game logic
2. **Login Server** (`login.js`) - Authentication and account management
3. **HTML5 Client** (`client/`) - Browser-based game client
4. **Game Data** (`data/740/`) - All game content definitions (items, monsters, NPCs, maps)

---

## 🏗️ Project Structure Overview

```
RPG-JS-Engine/
│
├── engine.js              # Main game server entry point
├── login.js               # Login server entry point
├── require.js             # Global module loader & configuration
├── config.json            # Server configuration
│
├── src/                   # Server-side source code (70+ files)
│   ├── gameserver.js      # Main server class
│   ├── world.js           # Game world container
│   ├── player.js          # Player entity
│   ├── creature.js        # Base creature class
│   ├── monster.js         # Monster entity
│   ├── npc.js             # NPC entity
│   ├── item.js            # Item class
│   ├── protocol.js        # Network packets
│   └── ...                # Many more modules
│
├── client/                # HTML5 game client
│   ├── index.html         # Main client page
│   ├── src/               # Client JavaScript (100+ files)
│   ├── css/               # Stylesheets
│   ├── png/               # UI images
│   └── sounds/            # Audio files
│
├── data/740/              # Game data for version 7.4
│   ├── items/             # Item definitions
│   ├── monsters/          # Monster definitions
│   ├── npcs/              # NPC definitions
│   ├── actions/           # Item action scripts
│   ├── spells/            # Spell definitions
│   ├── conditions/        # Status effect definitions
│   ├── world/             # Map files (.otbm)
│   └── ...                # More data folders
│
└── tools/                 # Development utilities
```

---

## 🔑 Key Concepts

### Server-Side Concepts

| Concept | Description |
|---------|-------------|
| **GameServer** | Main container that orchestrates all server components |
| **World** | Contains the game map (lattice), creatures, and events |
| **Creature** | Base class for all living entities (Player, Monster, NPC) |
| **Thing** | Base class for all objects (items, tiles, etc.) |
| **Chunk** | A section of the world map for efficient rendering |
| **Protocol** | Packet classes for client-server communication |

### Client-Side Concepts

| Concept | Description |
|---------|-------------|
| **GameClient** | Main client application container |
| **Renderer** | Canvas-based sprite rendering engine |
| **NetworkManager** | WebSocket communication handler |
| **World** | Client-side world representation |
| **Interface** | UI management and modals |

---

## 🚀 Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/your-repo/RPG-JS-Engine.git
cd RPG-JS-Engine

# 2. Install dependencies
npm install

# 3. Start all three processes (in separate terminals)
python client-server.py    # Static file server for client
node engine.js             # Game server
node login.js              # Login server

# 4. Open browser to http://127.0.0.1:8000/
# 5. Load Tibia.spr and Tibia.dat files (version 7.4)
# 6. Login with: Account: 111111, Password: tibia
```

---

## 📖 How to Read This Documentation

1. **New to the codebase?** Start with [02-ARCHITECTURE-OVERVIEW.md](./02-ARCHITECTURE-OVERVIEW.md)
2. **Want to understand entities?** Read [04-ENTITIES.md](./04-ENTITIES.md)
3. **Adding new content?** Check [10-ADDING-FEATURES.md](./10-ADDING-FEATURES.md)
4. **Need file details?** Use [11-FILE-REFERENCE.md](./11-FILE-REFERENCE.md)

---

## 🔧 Technology Stack

| Component | Technology |
|-----------|------------|
| Server Runtime | Node.js 16+ |
| Database | SQLite3 (accounts) |
| Networking | WebSocket (ws library) |
| Authentication | bcryptjs, HMAC-SHA256 |
| Client | Vanilla HTML5/JavaScript |
| Rendering | HTML5 Canvas |
| Map Format | OTBM (Open Tibia Binary Map) |
| Sprite Format | SPR/DAT (Tibia format) |

---

*Next: [Getting Started →](./01-GETTING-STARTED.md)*

