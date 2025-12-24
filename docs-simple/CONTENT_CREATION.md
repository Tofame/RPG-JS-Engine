# Content Creation Guide

This guide explains how to add new game content like monsters, items, and scripted actions to the engine.

## Directory Structure

All game data is located in `data/<version>/` (e.g., `data/740/`).

-   `actions/`: Scripted actions for items.
-   `items/`: Item definitions.
-   `monsters/`: Monster definitions.
-   `npcs/`: NPC definitions.
-   `spells/`: Spell definitions.
-   `world/`: The world map (OTBM).

## Adding a New Monster

1.  **Create the Definition File**:
    Create a new JSON file in `data/740/monsters/definitions/`, e.g., `dragon.json`.

    ```json
    {
      "creatureStatistics": {
        "name": "Dragon",
        "health": 1000,
        "maxHealth": 1000,
        "attack": 100,
        "defense": 30,
        "speed": 400,
        "outfit": {
          "id": 34
        }
      },
      "experience": 700,
      "corpse": 2844,
      "behaviour": {
        "type": 2,
        "fleeHealth": 0,
        "sayings": {
          "texts": ["GROOOAAAR"],
          "chance": 0.1
        }
      },
      "loot": [
        { "id": 2148, "min": 0, "max": 100, "probability": 1 }
      ]
    }
    ```

2.  **Register the Monster**:
    Edit `data/740/monsters/definitions.json` and add a mapping for your monster ID (must be unique).
    ```json
    {
      "1": "rat.json",
      "2": "dragon.json"
    }
    ```

## Adding a New Action (Script)

Actions allow you to define what happens when a player uses an item.

1.  **Create the Script**:
    Create a JS file in `data/740/actions/definitions/`, e.g., `myscript.js`.

    ```javascript
    module.exports = function(player, tile, index, item) {
      player.say("You used the custom item!");
      // Example: Remove the item
      tile.removeIndex(index, 1);
    };
    ```

2.  **Register the Action**:
    Edit `data/740/actions/definitions.json`.

    ```json
    [
      { "id": 1234, "on": "use", "callback": "myscript.js" }
    ]
    ```
    -   `id`: The Item ID to attach the script to.
    -   `on`: The event to listen for (`use`, `useWith`, `stepIn`, etc.).
    -   `callback`: The filename of your script.

## Adding Items

Items are currently loaded from `items.xml` and `items.otb` which are converted to JSON. To add new items, you typically need to:
1.  Edit `items.xml` (standard Open Tibia format).
2.  Run the conversion tool (if available in `tools/`) to regenerate the JSON definitions.

## Editing the Map

The server uses the `.otbm` format (Open Tibia Binary Map).
1.  Use a map editor like **RME (Remere's Map Editor)**.
2.  Save the map in the `7.4` format.
3.  Place the `.otbm` file in `data/740/world/`.
4.  Update `config.json` -> `WORLD` -> `WORLD_FILE` to point to your new map.
