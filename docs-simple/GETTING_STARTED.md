# Getting Started

## Prerequisites

-   **Node.js**: Version 16.0.0 or higher.
-   **Python**: For running the simple client file server (optional, can use other HTTP servers).
-   **Git**: To clone the repository.

## Installation

1.  **Clone the Repository**:
    ```bash
    git clone <repository-url>
    cd RPG-JS-Engine
    ```

2.  **Install Dependencies**:
    ```bash
    npm install
    ```
    This will install required packages like `ws`, `sqlite3`, `bcryptjs`, etc.

## Configuration

The main configuration file is `config.json` in the root directory.

### Key Configuration Options

-   **`SERVER`**:
    -   `HOST`: IP address to listen on (default `127.0.0.1`).
    -   `PORT`: Port for the game server (default `2222`).
    -   `MS_TICK_INTERVAL`: Game loop tick rate in ms (default `50`ms for 20 ticks/second).
    -   `CLIENT_VERSION`: The protocol version (default `740`).

-   **`LOGIN`**:
    -   `PORT`: Port for the login server (default `1337`).

-   **`WORLD`**:
    -   `WORLD_FILE`: The OTBM map file to load (e.g., `Tibia74.otbm`).
    -   `SPAWNS.ENABLED`: Set to `true` to enable monster spawns.
    -   `NPCS.ENABLED`: Set to `true` to enable NPCs.

-   **`DATABASE`**:
    -   `ACCOUNT_DATABASE`: SQLite file for accounts.

## Running the Server

You need to run three separate processes. You can do this in three separate terminal windows.

1.  **Client Server** (Serves the HTML/JS/Assets to the browser):
    ```bash
    python client-server.py
    ```
    *Access the client at `http://127.0.0.1:8000/`*

2.  **Login Server** (Handles authentication):
    ```bash
    node login.js
    ```

3.  **Game Server** (The main engine):
    ```bash
    node engine.js
    ```

## Logging In

1.  Open `http://127.0.0.1:8000/` in your browser.
2.  Use the default credentials (if configured in `config.json` under `DEFAULT_CHARACTER`):
    -   **Account**: `111111`
    -   **Password**: `tibia`

## Troubleshooting

-   **EADDRINUSE**: The port is already taken. Check if another instance is running or change the ports in `config.json`.
-   **Node Version**: Ensure you are running Node.js 16+. Check with `node -v`.
-   **Database Errors**: Ensure `sqlite3` built correctly. You might need to install build tools for your OS if `npm install` fails on sqlite3.
