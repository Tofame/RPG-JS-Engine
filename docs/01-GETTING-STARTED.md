# Getting Started

This guide will help you set up, run, and understand the basic operation of the RPG-JS-Engine.

---

## Prerequisites

Before you begin, ensure you have:

- **Node.js 16.0.0 or higher** - The server requires modern JavaScript features
- **Python 3.x** - For the simple static file server (or use any other HTTP server)
- **Tibia 7.4 Data Files** - You need `Tibia.spr` and `Tibia.dat` files (version 740)

---

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/Inconcessus/Tibia74-JS-Engine.git
cd Tibia74-JS-Engine
```

### Step 2: Install Dependencies

```bash
npm install
```

This installs the following packages:
- `ws` - WebSocket server
- `sqlite3` - SQLite database for accounts
- `bcryptjs` - Password hashing
- `jsonschema` - Data validation
- `xml2js` - XML parsing for data conversion

### Step 3: Configuration

The `config.json` file contains all server settings:

```json
{
  "HMAC": {
    "SHARED_SECRET": "your-secret-key-here"
  },
  "LOGIN": {
    "PORT": 1337,
    "HOST": "127.0.0.1",
    "TOKEN_VALID_MS": 3000
  },
  "SERVER": {
    "PORT": 2222,
    "HOST": "127.0.0.1",
    "CLIENT_VERSION": "740",
    "MS_TICK_INTERVAL": 50
  },
  "WORLD": {
    "WORLD_FILE": "Tibia74.otbm",
    "SPAWNS": { "ENABLED": false },
    "NPCS": { "ENABLED": false }
  }
}
```

Key settings to understand:

| Setting | Purpose |
|---------|---------|
| `HMAC.SHARED_SECRET` | Security key for token generation (change in production!) |
| `LOGIN.PORT` | Port for the login server (HTTP) |
| `SERVER.PORT` | Port for the game server (WebSocket) |
| `SERVER.CLIENT_VERSION` | Which data version to use (740 or 1098) |
| `SERVER.MS_TICK_INTERVAL` | Game loop tick rate (50ms = 20 ticks/second) |
| `WORLD.NPCS.ENABLED` | Enable/disable NPC spawning |
| `WORLD.SPAWNS.ENABLED` | Enable/disable monster spawning |

---

## Running the Server

You need to run **three separate processes**:

### Terminal 1: Static File Server (Client)

```bash
python client-server.py
```

This serves the HTML5 client at `http://127.0.0.1:8000/`

**Alternative:** You can use any HTTP server (nginx, Apache, `npx serve`, etc.)

### Terminal 2: Game Server

```bash
node engine.js
```

Output:
```
Starting NodeJS Forby Open Tibia Server
Creating server with version 740
Setting data directory to data/740
Loaded [[ X ]] items definitions.
Loaded [[ X ]] monsters definitions.
...
```

### Terminal 3: Login Server

```bash
node login.js
```

Output:
```
Starting NodeJS Forby Open Tibia Login Server.
The login server is listening for connections on 127.0.0.1:1337.
```

---

## First Login

### Step 1: Open the Client

Navigate to `http://127.0.0.1:8000/` in your browser.

### Step 2: Load Game Assets

1. Click **"Load Assets"**
2. Select both `Tibia.spr` and `Tibia.dat` files (version 7.4)
3. The status should change from "Missing" to "Loaded"

### Step 3: Login

Default credentials (from config.json):
- **Account Number:** `111111`
- **Password:** `tibia`

Click **"Enter Game"** and then **"Enter Gameworld"**

---

## Creating New Accounts

You can create accounts in two ways:

### Method 1: Client UI

1. Click **"Create Account"** on the login screen
2. Fill in account number, password, character name, and sex
3. Submit the form

### Method 2: HTTP API

```bash
curl -X POST "http://127.0.0.1:1337/?account=222222&password=mypass&name=newplayer&sex=male"
```

---

## In-Game Commands

Once logged in, use these commands in the chat:

| Command | Description |
|---------|-------------|
| `/waypoint <name>` | Teleport to a waypoint |
| `/goto <x> <y> <z>` | Teleport to coordinates |

Available waypoints:
- `rookgaard`, `thais`, `carlin`, `ab'dendriel`, `venore`
- `kazordoon`, `ankrahmun`, `edron`, `fibula`
- `gm-island`, `orc-fortress`, `cyclopolis`, `annihilator`

Example:
```
/waypoint thais
```

---

## Project File Structure

### Entry Points

| File | Purpose |
|------|---------|
| `engine.js` | Game server entry point |
| `login.js` | Login server entry point |
| `require.js` | Module loader and global setup |

### Source Directories

| Directory | Contents |
|-----------|----------|
| `src/` | Server-side modules (70+ files) |
| `client/src/` | Client-side modules (100+ files) |
| `client/css/` | Stylesheets |
| `client/png/` | UI graphics |
| `client/sounds/` | Audio files |

### Data Directories

| Directory | Contents |
|-----------|----------|
| `data/740/` | Game data for version 7.4 |
| `data/740/items/` | Item definitions |
| `data/740/monsters/` | Monster definitions |
| `data/740/npcs/` | NPC definitions |
| `data/740/spells/` | Spell scripts |
| `data/740/actions/` | Item action scripts |
| `data/740/world/` | Map file (OTBM) |

---

## Understanding the Module System

The project uses a custom module loading system defined in `require.js`:

### Global Functions

```javascript
// Load a module from src/
const Player = requireModule("player");

// Load a data file from data/740/
const monsterData = requireData("monsters", "definitions", "rat.json");

// Get the path to a data file
const mapPath = getDataFile("world", "Tibia74.otbm");
```

### Global Variables

```javascript
// Server configuration
global.CONFIG = require("./config");

// Game constants (loaded from client data)
global.CONST = require("./client/data/740/constants.json");

// Game server instance (after initialization)
global.gameServer = new GameServer();
```

---

## Development Workflow

### Making Changes

1. **Server changes:** Edit files in `src/`, restart `node engine.js`
2. **Client changes:** Edit files in `client/src/`, refresh browser
3. **Data changes:** Edit files in `data/740/`, restart game server
4. **Config changes:** Edit `config.json`, restart affected server

### Debugging

Enable verbose logging by checking the `CONFIG.LOGGING` settings:

```json
{
  "LOGGING": {
    "INTERVAL": 20,
    "NETWORK_TELEMETRY": true,
    "FILEPATH": "server.log"
  }
}
```

### Common Issues

| Issue | Solution |
|-------|----------|
| "Could not launch gameserver: required version > 16.0.0" | Upgrade Node.js |
| Client shows "Missing" for sprites | Load correct Tibia.spr file |
| "Connection refused" | Ensure all three servers are running |
| Account creation fails | Check login server logs |

---

## Next Steps

Now that you have the server running, learn about:

1. [Architecture Overview](./02-ARCHITECTURE-OVERVIEW.md) - Understand how components interact
2. [Server Core](./03-SERVER-CORE.md) - Deep dive into server modules
3. [Adding Features](./10-ADDING-FEATURES.md) - Create new game content

---

*Previous: [Index](./00-INDEX.md) | Next: [Architecture Overview →](./02-ARCHITECTURE-OVERVIEW.md)*

