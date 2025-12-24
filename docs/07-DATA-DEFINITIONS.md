# Data Definitions

This document explains how game data is structured and how to define monsters, NPCs, items, and other content.

---

## Table of Contents

1. [Data File Structure](#data-file-structure)
2. [Item Definitions](#item-definitions)
3. [Monster Definitions](#monster-definitions)
4. [NPC Definitions](#npc-definitions)
5. [Spell Definitions](#spell-definitions)
6. [Condition Definitions](#condition-definitions)

---

## Data File Structure

All game data is located in `data/740/` (or the configured version).

```
data/740/
├── items/
│   └── definitions.json       # All item definitions
├── monsters/
│   ├── definitions.json       # Monster ID mapping
│   └── definitions/           # Individual monster files
│       ├── rat.json
│       ├── skeleton.json
│       └── ...
├── npcs/
│   ├── definitions.json       # NPC spawn configuration
│   └── definitions/           # Individual NPC files
│       └── cipfried.json
├── spells/
│   ├── definitions.json       # Spell ID mapping
│   └── definitions/           # Spell scripts
│       ├── exura.js
│       └── fireball.js
├── conditions/
│   ├── definitions.json
│   └── definitions/
├── actions/
│   ├── definitions.json
│   └── definitions/
├── runes/
│   ├── definitions.json
│   └── definitions/
└── world/
    └── Tibia74.otbm           # Map file
```

### Definition Pattern

Most data types use a two-file pattern:

1. **definitions.json** - Maps IDs to file paths
2. **definitions/** folder - Contains the actual data files

```json
// monsters/definitions.json
{
  "1": "rat.json",
  "2": "skeleton.json",
  "3": "troll.json"
}
```

---

## Item Definitions

**File:** `data/740/items/definitions.json`

Items are defined in a single large JSON file.

### Basic Item

```json
{
  "2148": {
    "id": 3031,
    "group": 0,
    "flags": 8388608,
    "properties": {
      "name": "gold coin",
      "article": "a",
      "weight": 1,
      "type": "stackable"
    }
  }
}
```

### Item Properties

| Property | Description |
|----------|-------------|
| `id` | Client sprite ID |
| `flags` | Bitflags (pickupable, stackable, etc.) |
| `name` | Display name |
| `article` | "a", "an", or empty |
| `weight` | Weight in oz (×100 for real value) |
| `type` | Special type (container, door, etc.) |

### Special Types

```json
// Container
{
  "1988": {
    "properties": {
      "name": "backpack",
      "type": "container",
      "containerSize": 20
    }
  }
}

// Fluid Container
{
  "2006": {
    "properties": {
      "name": "vial",
      "type": "fluidContainer"
    }
  }
}

// Door
{
  "1209": {
    "properties": {
      "name": "door",
      "type": "door"
    }
  }
}

// Corpse (decaying container)
{
  "2813": {
    "properties": {
      "name": "dead rat",
      "type": "corpse",
      "containerSize": 4,
      "decayTo": 2814,
      "decayTime": 300000
    }
  }
}
```

---

## Monster Definitions

**File:** `data/740/monsters/definitions/[name].json`

### Complete Example

```json
{
  "creatureStatistics": {
    "name": "Rat",
    "health": 20,
    "maxHealth": 20,
    "mana": 0,
    "maxMana": 0,
    "attack": 8,
    "attackSpeed": 40,
    "defense": 5,
    "speed": 300,
    "outfit": {
      "id": 21
    }
  },
  "fluidType": 0,
  "experience": 5,
  "corpse": 2813,
  "behaviour": {
    "type": 2,
    "fleeHealth": 5,
    "openDoors": false,
    "senseInvisible": false,
    "sayings": {
      "texts": ["Meep!"],
      "slowness": 300,
      "chance": 0.75
    }
  },
  "loot": [
    {
      "id": 2148,
      "probability": 1,
      "min": 1,
      "max": 4
    },
    {
      "id": 2696,
      "probability": 0.1,
      "min": 1,
      "max": 1
    }
  ]
}
```

### Creature Statistics

| Field | Description |
|-------|-------------|
| `name` | Display name |
| `health` | Current health |
| `maxHealth` | Maximum health |
| `mana` | Current mana |
| `maxMana` | Maximum mana |
| `attack` | Attack damage |
| `attackSpeed` | Attack frequency |
| `defense` | Defense value |
| `speed` | Movement speed |
| `outfit` | Visual appearance |

### Behaviour Types

| Type | Behavior |
|------|----------|
| `0` | Passive - Flees when attacked |
| `1` | Defensive - Attacks only if attacked |
| `2` | Aggressive - Attacks on sight |

### Behaviour Options

| Field | Description |
|-------|-------------|
| `type` | Aggression type (0-2) |
| `fleeHealth` | HP at which monster flees |
| `openDoors` | Can open doors |
| `senseInvisible` | Can see invisible players |
| `sayings` | Random speech |

### Loot Table

```json
{
  "id": 2148,         // Item ID (gold coin)
  "probability": 1,    // 0-1 (100% = always drop)
  "min": 1,           // Minimum quantity
  "max": 4            // Maximum quantity
}
```

---

## NPC Definitions

**File:** `data/740/npcs/definitions/[name].json`

### Complete Example

```json
{
  "creatureStatistics": {
    "name": "Cipfried",
    "health": 50,
    "maxHealth": 50,
    "mana": 0,
    "maxMana": 0,
    "speed": 30,
    "outfit": {
      "id": 57,
      "details": {
        "head": 0,
        "body": 76,
        "legs": 116,
        "feet": 131
      }
    }
  },
  "behaviour": {
    "wanderRange": 2,
    "openDoors": true,
    "ignoreCharacters": true
  },
  "conversation": {
    "hearingRange": 5,
    "greetings": ["hello", "hi"],
    "farewells": ["bye"],
    "keywords": {
      "job": "I am a monk and the guardian of this temple.",
      "help": "I can heal you if you are wounded."
    },
    "trade": {
      "item": [
        {
          "name": "Shovel",
          "price": 5,
          "id": 2554,
          "type": "sell"
        },
        {
          "name": "Rope",
          "price": 5,
          "id": 2120,
          "type": "sell"
        }
      ]
    },
    "sayings": {
      "texts": ["Make sure to prepare!"],
      "rate": 300,
      "chance": 0.75
    },
    "script": "cipfried.js"
  }
}
```

### NPC Spawn Configuration

**File:** `data/740/npcs/definitions.json`

```json
{
  "1": {
    "definition": "cipfried.json",
    "enabled": true,
    "position": { "x": 32097, "y": 32205, "z": 7 }
  }
}
```

### Behaviour Options

| Field | Description |
|-------|-------------|
| `wanderRange` | Tiles to wander from spawn |
| `openDoors` | Can open doors |
| `ignoreCharacters` | Walk through players |

### Conversation Options

| Field | Description |
|-------|-------------|
| `hearingRange` | Tiles player must be within |
| `greetings` | Keywords to start conversation |
| `farewells` | Keywords to end conversation |
| `keywords` | Response map |
| `trade` | Shop items |
| `sayings` | Random ambient speech |
| `script` | Custom behavior script |

---

## Spell Definitions

**File:** `data/740/spells/definitions/[name].js`

### Spell Mapping

```json
// spells/definitions.json
{
  "1": "exura.js",
  "2": "fireball.js",
  "3": "haste.js"
}
```

### Spell Script Example

```javascript
// spells/definitions/exura.js
module.exports = function(player, spell) {
  
  // Check mana cost
  if(player.getProperty(CONST.PROPERTIES.MANA) < 20) {
    player.sendCancelMessage("Not enough mana.");
    return false;
  }
  
  // Consume mana
  player.incrementProperty(CONST.PROPERTIES.MANA, -20);
  
  // Heal player
  let healing = Math.floor(Math.random() * 50) + 30;
  player.incrementProperty(CONST.PROPERTIES.HEALTH, healing);
  
  // Show effect
  gameServer.world.sendMagicEffect(player.getPosition(), CONST.EFFECT.MAGIC_BLUE);
  
  return true;  // Spell cast successfully
}
```

---

## Condition Definitions

**File:** `data/740/conditions/definitions/[name].js`

### Condition Mapping

```json
// conditions/definitions.json
{
  "1": "poisoned.js",
  "2": "burning.js",
  "3": "haste.js"
}
```

### Condition Script Example

```javascript
// conditions/definitions/poisoned.js
module.exports = function(creature, condition) {
  
  // Called every tick while condition is active
  
  // Deal 1-5 poison damage
  let damage = Math.floor(Math.random() * 5) + 1;
  creature.decreaseHealth(null, damage);
  
  // Show effect
  gameServer.world.sendMagicEffect(
    creature.getPosition(),
    CONST.EFFECT.POISON
  );
  
  // Return false to remove condition
  return condition.numberTicks > 0;
}
```

---

## Summary

| Data Type | Location | Format |
|-----------|----------|--------|
| Items | `items/definitions.json` | Single JSON |
| Monsters | `monsters/definitions/` | Individual JSON |
| NPCs | `npcs/definitions/` | Individual JSON |
| Spells | `spells/definitions/` | JavaScript |
| Conditions | `conditions/definitions/` | JavaScript |
| Actions | `actions/definitions/` | JavaScript |
| Runes | `runes/definitions/` | JavaScript |

### Key Points

1. **IDs are server-side** - Client IDs are looked up from prototypes
2. **JSON for data** - Static properties like stats
3. **JS for behavior** - Dynamic logic like spells
4. **definitions.json** - Maps IDs to files
5. **Validation** - Use JSON schemas for type safety

---

*Previous: [Networking](./06-NETWORKING.md) | Next: [Actions & Events →](./08-ACTIONS-EVENTS.md)*

