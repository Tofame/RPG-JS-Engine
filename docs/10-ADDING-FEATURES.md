# Adding New Features

This guide explains how to add new content to the game: monsters, NPCs, items, spells, and more.

---

## Table of Contents

1. [Adding a Monster](#adding-a-monster)
2. [Adding an NPC](#adding-an-npc)
3. [Adding a Spell](#adding-a-spell)
4. [Adding an Item Action](#adding-an-item-action)
5. [Adding a Condition](#adding-a-condition)
6. [Adding a Rune](#adding-a-rune)

---

## Adding a Monster

### Step 1: Create Definition File

Create `data/740/monsters/definitions/wolf.json`:

```json
{
  "creatureStatistics": {
    "name": "Wolf",
    "health": 80,
    "maxHealth": 80,
    "mana": 0,
    "maxMana": 0,
    "attack": 15,
    "attackSpeed": 35,
    "defense": 10,
    "speed": 350,
    "outfit": {
      "id": 27
    }
  },
  "fluidType": 0,
  "experience": 25,
  "corpse": 2829,
  "behaviour": {
    "type": 2,
    "fleeHealth": 10,
    "openDoors": false,
    "senseInvisible": false,
    "sayings": {
      "texts": ["Grrrr!", "Bark!"],
      "slowness": 200,
      "chance": 0.5
    }
  },
  "loot": [
    {
      "id": 2148,
      "probability": 1,
      "min": 1,
      "max": 10
    },
    {
      "id": 2671,
      "probability": 0.5,
      "min": 1,
      "max": 2
    }
  ]
}
```

### Step 2: Register in Definitions

Add to `data/740/monsters/definitions.json`:

```json
{
  "1": "rat.json",
  "2": "skeleton.json",
  "20": "wolf.json"
}
```

### Step 3: Add Spawn (Optional)

Add to `data/740/spawns/definitions.json`:

```json
[
  {
    "monster": 20,
    "position": { "x": 32100, "y": 32200, "z": 7 },
    "radius": 5,
    "respawnTime": 60000,
    "count": 3
  }
]
```

### Step 4: Restart Server

```bash
node engine.js
```

---

## Adding an NPC

### Step 1: Create Definition File

Create `data/740/npcs/definitions/blacksmith.json`:

```json
{
  "creatureStatistics": {
    "name": "Tom the Blacksmith",
    "health": 100,
    "maxHealth": 100,
    "speed": 20,
    "outfit": {
      "id": 131,
      "details": {
        "head": 79,
        "body": 114,
        "legs": 132,
        "feet": 95
      }
    }
  },
  "behaviour": {
    "wanderRange": 1,
    "openDoors": false,
    "ignoreCharacters": true
  },
  "conversation": {
    "hearingRange": 4,
    "greetings": ["hello", "hi", "hey"],
    "farewells": ["bye", "farewell"],
    "keywords": {
      "job": "I am a blacksmith. I can repair your equipment.",
      "name": "My name is Tom. I've been smithing for 30 years.",
      "repair": "I can repair weapons and armor. Just show me what needs fixing.",
      "trade": "Sure, let me show you what I have for sale."
    },
    "trade": {
      "item": [
        { "name": "Sword", "price": 50, "id": 2376, "type": "sell" },
        { "name": "Mace", "price": 30, "id": 2398, "type": "sell" },
        { "name": "Steel Shield", "price": 75, "id": 2509, "type": "sell" }
      ]
    },
    "sayings": {
      "texts": [
        "Need anything repaired?",
        "*hammering sounds*",
        "Quality steel, straight from the forge!"
      ],
      "rate": 400,
      "chance": 0.6
    }
  }
}
```

### Step 2: Register in Definitions

Add to `data/740/npcs/definitions.json`:

```json
{
  "1": {
    "definition": "cipfried.json",
    "enabled": true,
    "position": { "x": 32097, "y": 32205, "z": 7 }
  },
  "2": {
    "definition": "blacksmith.json",
    "enabled": true,
    "position": { "x": 32150, "y": 32210, "z": 7 }
  }
}
```

### Step 3: Enable NPCs in Config

In `config.json`:

```json
{
  "WORLD": {
    "NPCS": {
      "ENABLED": true
    }
  }
}
```

---

## Adding a Spell

### Step 1: Create Spell Script

Create `data/740/spells/definitions/fireball.js`:

```javascript
module.exports = function(player, spell) {
  
  const MANA_COST = 30;
  const MIN_DAMAGE = 20;
  const MAX_DAMAGE = 40;
  
  // Check mana
  if(player.getProperty(CONST.PROPERTIES.MANA) < MANA_COST) {
    player.sendCancelMessage("Not enough mana.");
    return false;
  }
  
  // Get target
  let target = player.getTarget();
  if(!target) {
    player.sendCancelMessage("No target selected.");
    return false;
  }
  
  // Check range
  if(!player.isWithinRangeOf(target, 5)) {
    player.sendCancelMessage("Target is too far away.");
    return false;
  }
  
  // Check line of sight
  if(!player.isInLineOfSight(target)) {
    player.sendCancelMessage("Target is not in sight.");
    return false;
  }
  
  // Consume mana
  player.incrementProperty(CONST.PROPERTIES.MANA, -MANA_COST);
  
  // Calculate damage
  let damage = Math.floor(Math.random() * (MAX_DAMAGE - MIN_DAMAGE + 1)) + MIN_DAMAGE;
  
  // Apply damage
  target.decreaseHealth(player, damage);
  
  // Visual effects
  gameServer.world.sendDistanceEffect(
    player.getPosition(),
    target.getPosition(),
    CONST.DISTANCE.FIRE
  );
  
  gameServer.world.sendMagicEffect(
    target.getPosition(),
    CONST.EFFECT.FIRE_AREA
  );
  
  return true;
}
```

### Step 2: Register the Spell

Add to `data/740/spells/definitions.json`:

```json
{
  "1": "exura.js",
  "2": "fireball.js"
}
```

---

## Adding an Item Action

### Step 1: Create Action Script

Create `data/740/actions/definitions/fishing.js`:

```javascript
module.exports = function(player, item, tile) {
  
  // Check if target is water
  if(!tile || !isWaterTile(tile.id)) {
    player.sendCancelMessage("You can only fish in water.");
    return;
  }
  
  // Check fishing skill
  let fishingSkill = player.skills.getSkillLevel(CONST.PROPERTIES.FISHING);
  
  // Random chance based on skill
  let successChance = Math.min(0.1 + (fishingSkill * 0.01), 0.8);
  
  if(Math.random() < successChance) {
    // Caught a fish!
    let fish = gameServer.database.createThing(2667);  // Fish ID
    
    if(player.containerManager.equipment.canPushItem(fish)) {
      player.containerManager.equipment.pushItem(fish);
      player.sendServerMessage("You caught a fish!");
      
      // Increase fishing skill
      player.skills.addSkillPoints(CONST.PROPERTIES.FISHING, 1);
    } else {
      player.sendCancelMessage("You don't have room for the fish.");
    }
  } else {
    player.sendServerMessage("You didn't catch anything.");
  }
  
  // Play fishing effect
  gameServer.world.sendMagicEffect(
    tile.position,
    CONST.EFFECT.WATER_SPLASH
  );
}

function isWaterTile(id) {
  // Water tile IDs
  const WATER_TILES = [4608, 4609, 4610, 4611, 4612, 4613, 4614, 4615];
  return WATER_TILES.includes(id);
}
```

### Step 2: Register the Action

Add to `data/740/actions/definitions.json`:

```json
[
  {
    "on": "useWith",
    "id": 2580,
    "callback": "fishing.js"
  }
]
```

---

## Adding a Condition

### Step 1: Create Condition Script

Create `data/740/conditions/definitions/bleeding.js`:

```javascript
module.exports = function(creature, condition) {
  
  // Called every tick while condition is active
  
  // Deal damage (1-3)
  let damage = Math.floor(Math.random() * 3) + 1;
  creature.incrementProperty(CONST.PROPERTIES.HEALTH, -damage);
  
  // Show blood effect
  gameServer.world.sendMagicEffect(
    creature.getPosition(),
    CONST.EFFECT.BLOOD
  );
  
  // Create blood splash
  gameServer.world.addSplash(
    2016,  // Blood splash ID
    creature.getPosition(),
    CONST.FLUID.BLOOD
  );
  
  // Check if condition should continue
  condition.numberTicks--;
  
  // Return true to continue, false to remove
  return condition.numberTicks > 0;
}
```

### Step 2: Register the Condition

Add to `data/740/conditions/definitions.json`:

```json
{
  "1": "poisoned.js",
  "2": "burning.js",
  "10": "bleeding.js"
}
```

### Step 3: Apply the Condition

```javascript
// In a spell or item action
const Condition = requireModule("condition");

// Apply bleeding for 10 ticks, 2 seconds per tick
creature.conditions.add(
  new Condition(10, 10, 2000),  // id, ticks, interval
  null
);
```

---

## Adding a Rune

### Step 1: Create Rune Script

Create `data/740/runes/definitions/explosion-rune.js`:

```javascript
module.exports = function(player, rune, target) {
  
  const MANA_COST = 0;  // Runes don't cost mana
  const MIN_DAMAGE = 30;
  const MAX_DAMAGE = 60;
  const RADIUS = 2;
  
  // Get target position
  let targetPos = target.getPosition ? target.getPosition() : target;
  
  // Check range
  if(!player.getPosition().isWithinRangeOf(targetPos, 6)) {
    player.sendCancelMessage("Too far away.");
    return false;
  }
  
  // Get all creatures in radius
  let affectedCreatures = [];
  
  for(let x = -RADIUS; x <= RADIUS; x++) {
    for(let y = -RADIUS; y <= RADIUS; y++) {
      let checkPos = {
        x: targetPos.x + x,
        y: targetPos.y + y,
        z: targetPos.z
      };
      
      let tile = gameServer.world.getTileFromWorldPosition(checkPos);
      if(tile && tile.creatures) {
        tile.creatures.forEach(c => affectedCreatures.push(c));
      }
    }
  }
  
  // Deal damage to all affected creatures
  affectedCreatures.forEach(function(creature) {
    let damage = Math.floor(Math.random() * (MAX_DAMAGE - MIN_DAMAGE + 1)) + MIN_DAMAGE;
    creature.decreaseHealth(player, damage);
  });
  
  // Explosion effect
  gameServer.world.sendMagicEffect(targetPos, CONST.EFFECT.EXPLOSION);
  
  // Consume rune charge
  rune.useCharge();
  
  return true;
}
```

### Step 2: Register the Rune

Add to `data/740/runes/definitions.json`:

```json
{
  "1": "fireball.js",
  "2": "sudden-death.js",
  "10": "explosion-rune.js"
}
```

---

## Summary

### Workflow

1. **Create definition/script** in appropriate folder
2. **Register** in definitions.json
3. **Restart server** to load changes
4. **Test** in-game

### File Locations

| Feature | Definition File | Script Location |
|---------|-----------------|-----------------|
| Monster | `monsters/definitions/` | N/A (JSON only) |
| NPC | `npcs/definitions/` | `npcs/definitions/script/` |
| Spell | N/A | `spells/definitions/` |
| Action | `actions/definitions.json` | `actions/definitions/` |
| Condition | `conditions/definitions.json` | `conditions/definitions/` |
| Rune | `runes/definitions.json` | `runes/definitions/` |

### Testing Tips

1. Use `/waypoint` to teleport for testing
2. Check server console for errors
3. Add console.log for debugging
4. Use small respawn times during development

---

*Previous: [Client Overview](./09-CLIENT-OVERVIEW.md) | Next: [File Reference →](./11-FILE-REFERENCE.md)*

