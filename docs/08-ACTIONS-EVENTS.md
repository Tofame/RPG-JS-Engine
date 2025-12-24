# Actions and Events System

This document explains the action system for scripting item interactions and the event system for time-based logic.

---

## Table of Contents

1. [Action System Overview](#action-system-overview)
2. [Prototype Actions](#prototype-actions)
3. [Unique Actions](#unique-actions)
4. [Clock Events](#clock-events)
5. [Event Queue](#event-queue)
6. [Writing Action Scripts](#writing-action-scripts)

---

## Action System Overview

The action system allows scripting responses to player interactions with items.

### Action Types

| Type | Scope | Use Case |
|------|-------|----------|
| **Prototype Actions** | All items of a type | Using any shovel |
| **Unique Actions** | Single item instance | Special quest item |
| **Clock Events** | Time-based | Day/night changes |

### Action Loader

**File:** `src/database-action-loader.js`

```javascript
ActionLoader.prototype.initialize = function() {
  // Attach actions to item prototypes
  this.__attachPrototypeEvents("actions");
  
  // Load unique action definitions
  this.__attachUniqueEvents("unique");
}

ActionLoader.prototype.attachClockEvents = function(filepath) {
  // Attach time-based events
  gameServer.world.clock.on("time", callback);
}
```

---

## Prototype Actions

Actions that apply to ALL items of a certain type.

### Configuration

**File:** `data/740/actions/definitions.json`

```json
[
  {
    "on": "useWith",
    "id": 2554,
    "callback": "shovel.js"
  },
  {
    "on": "use",
    "from": 2674,
    "to": 2690,
    "callback": "food.js"
  },
  {
    "on": "useWith",
    "ids": [2553, 2554],
    "callback": "pick.js"
  }
]
```

### Configuration Options

| Field | Description |
|-------|-------------|
| `on` | Event type (`use`, `useWith`, `look`) |
| `id` | Single item ID |
| `ids` | Array of item IDs |
| `from/to` | Range of item IDs |
| `callback` | Script filename |

### Script Example: Shovel

```javascript
// data/740/actions/definitions/shovel.js
module.exports = function(player, tile) {
  
  // Get the tile being used on
  let targetTile = gameServer.world.getTileFromWorldPosition(tile.position);
  
  if(targetTile === null) {
    return player.sendCancelMessage("You cannot use this here.");
  }
  
  // Check if tile is diggable
  let groundId = targetTile.id;
  
  // Loose stone piles (can dig)
  if(groundId === 468 || groundId === 469) {
    // Transform to hole
    targetTile.transform(470);
    
    // Play effect
    gameServer.world.sendMagicEffect(
      tile.position,
      CONST.EFFECT.POFF
    );
    
    return;
  }
  
  player.sendCancelMessage("You cannot dig here.");
}
```

### Script Example: Food

```javascript
// data/740/actions/definitions/food.js
module.exports = function(player, item) {
  
  // Get food regeneration value
  let regeneration = getFoodValue(item.id);
  
  // Check if already full
  if(player.isSated(regeneration)) {
    return player.sendCancelMessage("You are full.");
  }
  
  // Consume the food
  item.removeCount(1);
  
  // Apply sated condition
  player.extendCondition(
    Condition.prototype.SATED,
    regeneration,
    1000  // Tick every second
  );
  
  // Success message
  player.sendServerMessage("Yum!");
}

function getFoodValue(id) {
  // Food IDs to regeneration ticks
  const FOOD_VALUES = {
    2674: 10,  // Apple
    2675: 15,  // Bread
    2689: 25,  // Ham
  };
  return FOOD_VALUES[id] || 5;
}
```

---

## Unique Actions

Actions that apply to a SINGLE specific item instance (by action ID).

### Configuration

**File:** `data/740/unique/definitions.json`

```json
[
  {
    "on": "use",
    "uid": 1001,
    "callback": "clock-secret.js"
  },
  {
    "on": "stepIn",
    "uid": 1002,
    "callback": "enter-trunk.js"
  }
]
```

### Event Types

| Event | Trigger |
|-------|---------|
| `use` | Player uses the item |
| `useWith` | Player uses something on the item |
| `look` | Player looks at the item |
| `stepIn` | Player steps onto the tile |
| `stepOut` | Player steps off the tile |

### Script Example: Secret Lever

```javascript
// data/740/unique/definitions/clock-secret.js
module.exports = function(player, tile, thing) {
  
  // Check if lever is already pulled
  if(thing.id === 1946) {  // Pulled lever
    player.sendCancelMessage("The lever is stuck.");
    return;
  }
  
  // Pull the lever
  thing.transform(1946);
  
  // Open secret passage
  let passageTile = gameServer.world.getTileFromWorldPosition({
    x: thing.getPosition().x + 2,
    y: thing.getPosition().y,
    z: thing.getPosition().z
  });
  
  if(passageTile) {
    passageTile.transform(469);  // Open hole
  }
  
  // Play sound effect
  gameServer.world.sendMagicEffect(
    thing.getPosition(),
    CONST.EFFECT.POFF
  );
  
  // Schedule lever reset
  gameServer.world.eventQueue.addEvent(function() {
    thing.transform(1945);  // Reset lever
    passageTile.transform(468);  // Close passage
  }, 60000, null);  // Reset after 1 minute
}
```

---

## Clock Events

Events triggered by the game world clock (time of day).

### Configuration

**File:** `data/740/clock/definitions.json`

```json
[
  { "callback": "timetile.js" },
  { "callback": "cemetery-ghost.js" }
]
```

### Script Example: Time-based Tile

```javascript
// data/740/clock/definitions/timetile.js
module.exports = function(timeString) {
  
  // Parse time
  let [hours, minutes] = timeString.split(":").map(Number);
  
  // Night time: 20:00 - 06:00
  let isNight = hours >= 20 || hours < 6;
  
  // Transform tiles based on time
  TIME_TILES.forEach(function(config) {
    let tile = gameServer.world.getTileFromWorldPosition(config.position);
    
    if(tile === null) return;
    
    if(isNight) {
      tile.transform(config.nightId);
    } else {
      tile.transform(config.dayId);
    }
  });
}

const TIME_TILES = [
  { position: { x: 1000, y: 1000, z: 7 }, dayId: 100, nightId: 101 }
];
```

### Script Example: Midnight Spawns

```javascript
// data/740/clock/definitions/cemetery-ghost.js
module.exports = function(timeString) {
  
  // Only at midnight
  if(timeString !== "00:00") return;
  
  // Spawn ghost at cemetery
  let ghost = gameServer.database.createMonster("ghost");
  
  gameServer.world.creatureHandler.addCreatureSpawn(
    ghost,
    { x: 32150, y: 32200, z: 7 }
  );
  
  // Remove after 1 hour
  gameServer.world.eventQueue.addEvent(function() {
    gameServer.world.creatureHandler.removeCreature(ghost);
  }, 3600000, null);
}
```

---

## Event Queue

The event queue schedules future actions.

### Basic Usage

```javascript
// Schedule event for 5 seconds from now
let event = gameServer.world.eventQueue.addEvent(
  callback,     // Function to call
  5000,         // Delay in milliseconds
  context       // 'this' context for callback
);

// Cancel a scheduled event
gameServer.world.eventQueue.cancelEvent(event);
```

### Common Patterns

```javascript
// Item decay
item.decayEvent = world.eventQueue.addEvent(
  item.decay.bind(item),
  item.getDecayTime(),
  item
);

// Condition tick
condition.tickEvent = world.eventQueue.addEvent(
  condition.tick.bind(condition),
  condition.tickInterval,
  condition
);

// Respawn timer
spawn.respawnEvent = world.eventQueue.addEvent(
  spawn.respawnMonster.bind(spawn),
  spawn.respawnTime,
  spawn
);
```

---

## Writing Action Scripts

### Script Parameters

| Event | Parameters |
|-------|------------|
| `use` | `(player, item)` |
| `useWith` | `(player, item, targetItem)` |
| `look` | `(player, item)` |
| `stepIn` | `(player, tile)` |
| `stepOut` | `(player, tile, newPosition)` |

### Common Operations

```javascript
// Send message to player
player.sendCancelMessage("You cannot do this.");
player.sendServerMessage("Success!");

// Modify items
item.setCount(10);
item.removeCount(1);
item.transform(newId);

// Modify tiles
tile.transform(newGroundId);
tile.addTopThing(item);

// Effects
gameServer.world.sendMagicEffect(position, CONST.EFFECT.POFF);
gameServer.world.sendDistanceEffect(from, to, CONST.DISTANCE.ARROW);

// Teleport player
gameServer.world.creatureHandler.teleportCreature(player, newPosition);

// Create items
let item = gameServer.database.createThing(itemId);
item.setCount(5);
player.containerManager.equipment.pushItem(item);

// Schedule events
gameServer.world.eventQueue.addEvent(callback, delay, context);
```

### Return Values

```javascript
// Return nothing or undefined: action completed normally

// Return false: action failed, can be retried

// Throw error: action failed with error
```

---

## Summary

| System | Purpose | Configuration |
|--------|---------|---------------|
| Prototype Actions | All items of type | `actions/definitions.json` |
| Unique Actions | Single item | `unique/definitions.json` |
| Clock Events | Time-based | `clock/definitions.json` |
| Event Queue | Scheduled events | Programmatic |

### Best Practices

1. **Check preconditions** - Validate player state before action
2. **Send feedback** - Use cancel/server messages
3. **Clean up** - Cancel events when no longer needed
4. **Use constants** - Reference CONST for IDs and effects
5. **Handle errors** - Try/catch for robustness

---

*Previous: [Data Definitions](./07-DATA-DEFINITIONS.md) | Next: [Client Overview →](./09-CLIENT-OVERVIEW.md)*

