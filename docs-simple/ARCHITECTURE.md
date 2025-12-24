# System Architecture

## Overview

The RPG-JS-Engine uses a monolithic architecture separated into three processes for scalability and separation of concerns:
1.  **Client (Browser)**: Handles rendering and input.
2.  **Login Server**: Authenticates users and issues tokens.
3.  **Game Server**: Runs the game simulation and handles persistent connections.

## The Game Loop (`src/gameloop.js`)

The server runs on a fixed time-step loop.
-   **Interval**: Configured via `MS_TICK_INTERVAL` (default 50ms).
-   **Drift Correction**: The loop calculates the execution time and drift to adjust the timeout for the next tick, ensuring a stable tick rate.
-   **Execution**:
    1.  `GameServer.__loop()` is called.
    2.  `HTTPServer` flushes socket buffers.
    3.  `World.tick()` is executed.

## World Management (`src/world.js`, `src/lattice.js`)

The world is managed by the `World` class, but the spatial storage is handled by the `Lattice`.

### Lattice & Chunks
The world is divided into 3D blocks called **Chunks** (or Sectors).
-   **Chunk Size**: Configurable (Default 9x7x8).
-   **Active Chunks**: Only chunks with players (and their neighbours) are considered "active" for processing certain events.
-   **Spectating**: Players "spectate" their surrounding chunks. Changes in a chunk are broadcast to all spectating players.

### Tiles & Things
-   **Tile**: Represents a single coordinate (x, y, z). Contains a stack of `Things`.
-   **Thing**: The base class for everything in the game (Items, Creatures).
-   **Stacking**: Tiles manage a stack of items. The top item is usually the one interacted with.

## Networking

### Protocol
The communication uses **WebSockets** with a custom binary protocol.
-   **Handshake**:
    1.  Client connects via HTTP to `Login Server`.
    2.  Login Server verifies credentials and returns a session **Token**.
    3.  Client connects via HTTP Upgrade to `Game Server` with the token.
    4.  `Game Server` validates token (via shared secret HMAC) and upgrades to WebSocket.

### Packet Handling (`src/packet-handler.js`)
Incoming binary data is wrapped in `PacketReader`.
The `PacketReader` parses the opcodes and data types.
The `PacketHandler` routes the parsed data to the appropriate logic methods (e.g., `handleMoveItem`, `handlePlayerSay`).

## Entity System

The class hierarchy for game objects:

-   **`Thing`** (`src/thing.js`)
    -   **`Item`** (`src/item.js`)
        -   `Container`, `Equipment`, `Teleporter`, `Door`, etc.
    -   **`Creature`** (`src/creature.js`)
        -   **`Player`** (`src/player.js`)
        -   **`Monster`** (`src/monster.js`)
        -   **`NPC`** (`src/npc.js`)

`Creature` handles:
-   Properties (Health, Mana, Speed).
-   Conditions (Poison, Drunk).
-   Position and Movement.
-   Outfit.

## Data Loading (`src/database.js`)

The `Database` class is responsible for loading all static assets at startup:
1.  **Items**: Mapped from `items.xml` + `items.otb` (merged to JSON).
2.  **Map**: OTBM binary file parsed by `OTBMParser`.
3.  **JSON Definitions**:
    -   Monsters, Spells, Runes, NPCs, Houses.
    -   Actions (Scripts attached to items/events).

## Action System (`src/database-action-loader.js`)

The engine allows attaching JavaScript callbacks to game events via JSON configuration.
-   **Unique Actions**: Attached to a specific item `uid`.
-   **Prototype Actions**: Attached to an item `id` (all items of that type).
-   **Events**: `use`, `useWith`, `move`, etc.
