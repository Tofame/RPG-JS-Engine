# Code Guide

This guide is for developers who want to modify the core engine code in `src/`.

## Key Classes

### `GameServer` (`src/gameserver.js`)
The root object. Accessible globally via `gameServer`.
-   `gameServer.world`: Access the world state.
-   `gameServer.database`: Access loaded data.
-   `gameServer.server`: Access the HTTP/WebSocket server.

### `World` (`src/world.js`)
Manages the global state.
-   `world.tick()`: Called every frame.
-   `world.lattice`: The map structure (chunks/tiles).
-   `world.creatureHandler`: Manages active creatures.
-   `world.broadcastPacket()`: Send data to all players.

### `Creature` (`src/creature.js`)
Base class for all living things.
-   `properties`: Health, mana, speed, direction.
-   `conditions`: Status effects (poison, drunk).
-   `position`: Current `Position(x, y, z)`.

### `Player` (`src/player.js`)
Extends `Creature`. Represents a connected user.
-   `socket`: The WebSocket connection.
-   `containerManager`: Manages opened bags/backpacks.
-   `skills`: Experience and skills.
-   `write(packet)`: Send a packet to this player.

## Packet Handling Flow

1.  **Ingest**: `WebSocketServer` receives binary data.
2.  **Route**: `Player.readPacket()` is called.
3.  **Parse**: `PacketReader` reads the opcode.
4.  **Handle**: `PacketHandler` (in `src/packet-handler.js`) switches on the opcode and calls the appropriate method (e.g., `handleMoveItem`).

## Adding a New Network Packet

To add a new feature that requires client-server communication:

1.  **Define Opcode**: Choose a new byte for your packet ID.
2.  **Client Side**: Implement the writer/reader in the client code.
3.  **Server Reader**: Add a method to `PacketReader` (`src/packet-reader.js`) to parse the data.
4.  **Server Handler**: Add a case in `PlayerSocketHandler` (or `PacketHandler`) to route the new opcode.
5.  **Server Logic**: Implement the logic in `PacketHandler`.

## Global Helpers

-   `CONFIG`: Access `config.json` values.
-   `CONST`: Access constants from `client/data/740/constants.json`.
-   `getDataFile(...)`: Resolve paths to `data/` directory.
