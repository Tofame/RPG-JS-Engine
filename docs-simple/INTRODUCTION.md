# Introduction to RPG-JS-Engine

Welcome to the RPG-JS-Engine documentation. This project is a custom Open Tibia 7.4 server implementation written in Node.js (Server) and utilizing a custom HTML5 Client.

## Core Design

The engine is designed to be modular and easy to extend using JavaScript. It departs from the traditional C++ Open Tibia servers (TFS) by using a JavaScript-based game loop and logic, making it more accessible for web developers.

## Key Features

-   **Node.js Server**: Built on top of Node.js for event-driven, non-blocking I/O.
-   **WebSocket Protocol**: Uses WebSockets for real-time communication between client and server.
-   **HTML5 Client**: A browser-based client (no download required for players).
-   **Custom Binary Protocol**: Efficient binary protocol over WebSockets.
-   **OTBM Map Support**: Supports standard Open Tibia Binary Map (OTBM) format (version 7.4).
-   **Scriptable**: Extensive use of JSON and JavaScript for defining items, monsters, NPCs, and game logic (actions).
-   **Lattice Spatial Partitioning**: efficient map management using chunks.

## Project Structure Overview

-   `engine.js`: The main game server entry point.
-   `login.js`: The login/authentication server.
-   `client-server.py`: A simple HTTP server to serve the client files (can be replaced by Nginx/Apache).
-   `src/`: Contains the core server source code (classes, handlers, logic).
-   `data/`: Contains game data (maps, items, monsters, scripts).
-   `client/`: Contains the HTML5 client source code.

## Next Steps

-   **[Getting Started](GETTING_STARTED.md)**: Learn how to install and run the server.
-   **[Architecture](ARCHITECTURE.md)**: Understand the system design, game loop, and networking.
-   **[Content Creation](CONTENT_CREATION.md)**: Learn how to add new items, monsters, and write scripts.
-   **[Code Guide](CODE_GUIDE.md)**: Deep dive into the codebase and core classes.
