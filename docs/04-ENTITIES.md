# Entities: Creatures, Players, Monsters, and NPCs

This document explains the entity hierarchy and how each entity type works in the game engine.

---

## Table of Contents

1. [Entity Hierarchy](#entity-hierarchy)
2. [Creature (Base Class)](#creature-base-class)
3. [Player](#player)
4. [Monster](#monster)
5. [NPC](#npc)
6. [Creature Handler](#creature-handler)

---

## Entity Hierarchy

All living entities inherit from a common base:

```
EventEmitter (src/eventemitter.js)
    │
    └── Creature (src/creature.js)
            │
            ├── Player (src/player.js)
            │
            ├── Monster (src/monster.js)
            │
            └── NPC (src/npc.js)
```

### Why EventEmitter?

The EventEmitter pattern allows loose coupling:

```javascript
// A creature can emit events
creature.emit("death");
creature.emit("move", oldPosition, newPosition);
creature.emit("damage", source, amount);

// Other systems can listen without tight coupling
creature.on("death", function() {
  this.lootHandler.addLoot(corpse);
  this.damageMap.distributeExperience();
});
```

---

## Creature (Base Class)

**File:** `src/creature.js`

The Creature class contains common functionality for all living entities.

### Constructor

```javascript
const Creature = function(properties) {
  // Inherit from EventEmitter
  EventEmitter.call(this);
  
  // All creatures start with no position (set on spawn)
  this.position = null;
  
  // Properties container (health, mana, speed, etc.)
  this.properties = new CreatureProperties(this, properties);
  
  // Active conditions (poison, haste, etc.)
  this.conditions = new ConditionManager(this);
  
  // Speech handler for saying things
  this.speechHandler = new SpeechHandler(this);
}
```

### Properties System

```javascript
// Get a property value
creature.getProperty(CONST.PROPERTIES.HEALTH);
creature.getProperty(CONST.PROPERTIES.SPEED);

// Set a property value
creature.setProperty(CONST.PROPERTIES.HEALTH, 100);

// Increment a property (for damage/healing)
creature.incrementProperty(CONST.PROPERTIES.HEALTH, -10);

// Check if at max value
creature.isFull(CONST.PROPERTIES.HEALTH);  // true if health == maxHealth
```

### Position & Movement

```javascript
// Get current position
let pos = creature.getPosition();  // Position { x, y, z }

// Set position (used by teleport)
creature.setPosition(newPosition);

// Get current chunk
let chunk = creature.getChunk();

// Get tile creature is standing on
let tile = creature.getTile();

// Check if beside another thing
creature.isBesidesThing(item);

// Check if can see a position
creature.canSee(targetPosition);

// Check line of sight
creature.isInLineOfSight(otherCreature);
```

### Conditions

```javascript
// Check if has a condition
creature.hasCondition(Condition.prototype.POISONED);
creature.hasCondition(Condition.prototype.HASTE);

// Remove a condition
creature.removeCondition(Condition.prototype.INVISIBLE);

// Check drunk state
creature.isDrunk();
```

### Outfit

```javascript
// Get current outfit
let outfit = creature.getOutfit();  // { id, head, body, legs, feet, mounted, addons }

// Change outfit
creature.changeOutfit(newOutfit);

// Check if mounted
creature.isMounted();
```

### Combat Properties

```javascript
// Calculate random damage dealt
let damage = creature.calculateDamage();  // 0 to attack value

// Calculate random defense
let blocked = creature.calculateDefense();  // 0 to defense value

// Get attack/defense values
creature.getAttack();
creature.getDefense();
creature.getAttackSpeed();
```

### Broadcasting

```javascript
// Broadcast packet to all observers
creature.broadcast(new SomePacket(...));

// Broadcast to same floor only
creature.broadcastFloor(packet);
```

### Movement Speed

```javascript
// Calculate step duration based on speed and friction
let duration = creature.getStepDuration(tileFriction);

// Formula: See https://tibia.fandom.com/wiki/Speed_Breakpoints
```

---

## Player

**File:** `src/player.js`

The Player class extends Creature with player-specific functionality.

### Constructor

```javascript
const Player = function(data) {
  // Inherit from Creature
  Creature.call(this, data.properties);
  
  // Respawn position
  this.templePosition = Position.prototype.fromLiteral(data.templePosition);
  
  // Add player-specific properties
  this.addPlayerProperties(data.properties);
  
  // Character skills (experience, magic, combat skills)
  this.skills = new Skills(this, data.skills);
  
  // Network communication
  this.socketHandler = new SocketHandler(this);
  
  // Friend list
  this.friendlist = new Friendlist(data.friends);
  
  // Inventory and equipment
  this.containerManager = new ContainerManager(this, data.containers);
  
  // Learned spells
  this.spellbook = new Spellbook(this, data.spellbook);
  
  // Idle timeout management
  this.idleHandler = new PlayerIdleHandler(this);
  
  // Walking logic
  this.movementHandler = new PlayerMovementHandler(this);
  
  // Chat channel management
  this.channelManager = new ChannelManager(this);
  
  // Action processing (use, look, attack)
  this.actionHandler = new ActionHandler(this);
  
  // Combat state
  this.combatLock = new CombatLock(this);
  
  // Item usage
  this.useHandler = new UseHandler(this);
}
```

### Player-Specific Properties

```javascript
// Vocation (sorcerer, druid, paladin, knight, none)
player.getVocation();

// Level
player.getLevel();

// Experience points
player.getExperiencePoints();

// Carrying capacity
player.getCapacity();

// Check if has capacity for item
player.hasSufficientCapacity(item);

// Text color (yellow for normal, red for admin)
player.getTextColor();
```

### Equipment & Inventory

```javascript
// Check if distance weapon equipped
player.isDistanceWeaponEquipped();

// Check if ammunition available
player.isAmmunitionEquipped();

// Consume one arrow/bolt
player.consumeAmmunition();

// Get equipment attribute sum
player.getEquipmentAttribute("armor");
player.getEquipmentAttribute("attack");
```

### Containers

```javascript
// Open a container
player.openContainer(id, name, baseContainer);

// Close a container
player.closeContainer(baseContainer);
```

### Combat

```javascript
// Check combat state
player.isInCombat();  // Returns true if recently attacked/attacked

// Get attack value (based on weapon, skills, mode)
player.getAttack();

// Get defense value
player.getDefense();

// Get base damage from level
player.getBaseDamage();  // floor(level / 5)

// Handle taking damage
player.decreaseHealth(source, amount);

// Handle death
player.handleDeath();  // Teleport to temple, restore health
```

### Conditions

```javascript
// Add or extend a condition
player.extendCondition(id, ticks, duration);

// Check if sated (for food consumption)
player.isSated(ticks);

// Check invisibility
player.isInvisible();
```

### Location Checks

```javascript
// Check protection zone
player.isInProtectionZone();

// Check no-logout zone
player.isInNoLogoutZone();

// Check house ownership
player.ownsHouseTile(tile);
```

### Network Communication

```javascript
// Write packet to player's socket
player.write(new SomePacket(...));

// Send cancel message (red text)
player.sendCancelMessage("You cannot do that.");

// Disconnect player
player.disconnect();
```

### Serialization (Saving)

```javascript
// Convert player to JSON for saving
player.toJSON();
// Returns: { position, skills, properties, containers, spellbook, friends, ... }
```

### Cleanup (Logout)

```javascript
player.cleanup();  // Called on logout
// - Leave all channels
// - Close all containers
// - Cancel condition events
// - Disconnect socket
// - Emit "logout" event
```

---

## Monster

**File:** `src/monster.js`

Monsters are hostile creatures with AI behavior.

### Constructor

```javascript
const Monster = function(cid, data) {
  // Inherit from Creature
  Creature.call(this, data.creatureStatistics);
  
  // Creature type identifier (for respawn)
  this.cid = cid;
  
  // Corpse item ID
  this.corpse = data.corpse;
  
  // Blood color
  this.fluidType = CONST.COLOR.RED;
  
  // Experience reward
  this.experience = data.experience;
  
  // Track damage sources for experience distribution
  this.damageMap = new DamageMap(this);
  
  // Loot table
  this.lootHandler = new LootHandler(data.loot);
  
  // AI behavior (chase, flee, wander)
  this.behaviourHandler = new MonsterBehaviour(this, data.behaviour);
}
```

### Monster Definition (JSON)

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
    "outfit": { "id": 21 }
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
    { "id": 2148, "probability": 1, "min": 1, "max": 4 },
    { "id": 2696, "probability": 0.1, "min": 1, "max": 1 }
  ]
}
```

### AI Behavior

```javascript
// The monster "thinks" every tick
Monster.prototype.think = function() {
  this.behaviourHandler.actions.handleActions(this.behaviourHandler);
}

// Target management
monster.setTarget(player);
monster.getTarget();
monster.hasTarget();

// Tile occupation check (for pathfinding)
monster.isTileOccupied(tile);  // Can't enter protection zones
```

### Behavior Types

| Type | Description |
|------|-------------|
| 0 | Passive - Wanders, flees when attacked |
| 1 | Defensive - Attacks only when attacked |
| 2 | Aggressive - Attacks players on sight |

### Combat

```javascript
// Handle receiving damage
monster.decreaseHealth(source, amount);
// - Records in damageMap
// - Updates health
// - Triggers flee behavior if low health
// - Dies if health reaches 0

// Create corpse on death
monster.createCorpse();
// - Creates corpse item
// - Distributes experience to attackers
// - Adds loot to corpse
```

### Loot System

```javascript
// Loot is defined as probability-based drops
{
  "id": 2148,        // Gold coin
  "probability": 1,   // 100% chance
  "min": 1,          // Minimum amount
  "max": 4           // Maximum amount
}

// Loot is added to corpse container
this.lootHandler.addLoot(corpse);
```

### Spell Actions (for magic-using monsters)

```javascript
// Monsters can have spell lists
monster.handleSpellAction();
// - Checks cooldowns
// - Checks line of sight
// - Casts available spells based on chance
```

---

## NPC

**File:** `src/npc.js`

NPCs are non-hostile creatures for quests, trading, and conversation.

### Constructor

```javascript
const NPC = function(data) {
  // Inherit from Creature
  Creature.call(this, data.creatureStatistics);
  
  // Conversation system
  this.conversationHandler = new ConversationHandler(this, data.conversation);
  
  // Movement behavior
  this.behaviourHandler = new NPCBehaviour(this, data.behaviourHandler);
  
  // Scripted sequences
  this.cutsceneHandler = new CutsceneHandler(this);
  
  // Available actions (wander, speak)
  this.actions = new Actions();
  
  this.__registerActions();
}
```

### NPC Definition (JSON)

```json
{
  "creatureStatistics": {
    "name": "Cipfried",
    "health": 50,
    "maxHealth": 50,
    "speed": 30,
    "outfit": {
      "id": 57,
      "details": { "head": 0, "body": 76, "legs": 116, "feet": 131 }
    }
  },
  "behaviour": {
    "wanderRange": 2,
    "openDoors": true,
    "ignoreCharacters": true
  },
  "conversation": {
    "hearingRange": 5,
    "keywords": { "test": "This is a test response." },
    "trade": {
      "item": [
        { "name": "Shovel", "price": 5, "id": 2554, "type": "sell" },
        { "name": "Rope", "price": 5, "id": 2120, "type": "sell" }
      ]
    },
    "greetings": ["hello", "hi"],
    "farewells": ["bye"],
    "sayings": {
      "texts": ["Make sure to prepare!"],
      "rate": 300,
      "chance": 0.75
    },
    "script": "cipfried.js"
  }
}
```

### Conversation System

```javascript
// NPC listens to nearby player messages
NPC.prototype.listen = function(player, message) {
  // Skip if in cutscene
  if(this.cutsceneHandler.isInScene()) return;
  
  // Check if player is in range
  if(!this.isWithinHearingRange(player)) return;
  
  // Handle the message
  this.conversationHandler.handleResponse(player, message);
}

// Check if NPC is talking to someone
npc.isInConversation();

// Hearing range check
npc.isWithinHearingRange(creature);
```

### Actions

```javascript
// Wandering action
NPC.prototype.handleActionWander = function() {
  let tile = this.behaviourHandler.getWanderMove();
  if(tile !== null) {
    gameServer.world.creatureHandler.moveCreature(this, tile.position);
  }
  this.actions.lock(this.handleActionWander, duration);
}

// Speaking action (random sayings)
NPC.prototype.handleActionSpeak = function() {
  let sayings = this.conversationHandler.getSayings();
  if(Math.random() > (1.0 - sayings.chance)) {
    this.speechHandler.internalCreatureSay(sayings.texts.random());
  }
  this.actions.lock(this.handleActionSpeak, sayings.rate);
}
```

### NPC Think Loop

```javascript
NPC.prototype.think = function() {
  // Don't process if in conversation
  if(this.isInConversation()) return;
  
  // Don't process if in cutscene
  if(this.cutsceneHandler.isInScene()) return;
  
  // Handle wander/speak actions
  this.actions.handleActions(this);
}
```

### Cutscenes

```javascript
// Set NPC to perform a scripted sequence
npc.setScene(scene);
// - Aborts current cutscene if any
// - Drops conversation if any
// - Plays scripted actions

// Pause NPC actions briefly
npc.pauseActions(duration);
```

---

## Creature Handler

**File:** `src/world-creature-handler.js`

The CreatureHandler manages all creatures in the world.

### Key Functions

```javascript
// Get all connected players
creatureHandler.getConnectedPlayers();

// Check if player is online
creatureHandler.isPlayerOnline(player);

// Get creature by unique ID
creatureHandler.getCreatureFromId(id);

// Add creature to the world
creatureHandler.addCreatureSpawn(creature, position);

// Move a creature
creatureHandler.moveCreature(creature, newPosition);

// Teleport a creature
creatureHandler.teleportCreature(creature, newPosition);

// Handle creature death
creatureHandler.dieCreature(creature);

// Process all creatures each tick
creatureHandler.tick();
```

### Creature Tracking

```javascript
// All creatures have a unique ID
creature.getId();  // Returns globally unique identifier

// The handler maintains maps for lookup
this.__creatures = new Map();      // ID → Creature
this.__players = new Map();        // Name → Player
this.__monsters = new Set();       // Active monsters
this.__npcs = new Set();           // Active NPCs
```

### Tick Processing

```javascript
CreatureHandler.prototype.tick = function() {
  // Process each active creature
  this.__activeCreatures.forEach(function(creature) {
    creature.think();  // Let creature decide actions
  });
}
```

---

## Summary

| Entity | Purpose | AI | Combat | Trading |
|--------|---------|-----|--------|---------|
| **Creature** | Base class | No | Properties only | No |
| **Player** | Human player | User input | Full combat | With NPCs |
| **Monster** | Enemy AI | Chase/Flee | Attacks players | Drops loot |
| **NPC** | Friendly AI | Wander/Talk | Usually none | Trading system |

### Key Patterns

1. **Inheritance** - All entities extend Creature
2. **Composition** - Handlers add specific functionality
3. **Events** - Loose coupling via EventEmitter
4. **Think Loop** - Each entity processes actions each tick
5. **Properties** - Unified system for stats like health, speed

---

*Previous: [Server Core](./03-SERVER-CORE.md) | Next: [Items & Containers →](./05-ITEMS-CONTAINERS.md)*

