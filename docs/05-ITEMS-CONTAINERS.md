# Items, Containers, and Equipment

This document explains the item system, containers, and player equipment.

---

## Table of Contents

1. [Item Hierarchy](#item-hierarchy)
2. [Thing (Base Class)](#thing-base-class)
3. [Item](#item)
4. [Specialized Item Types](#specialized-item-types)
5. [Containers](#containers)
6. [Equipment System](#equipment-system)
7. [Item Stacks on Tiles](#item-stacks-on-tiles)

---

## Item Hierarchy

```
EventEmitter
    │
    └── Thing (src/thing.js)
            │
            ├── Item (src/item.js)
            │
            ├── Container (src/container.js)
            │   └── Corpse (src/corpse.js)
            │
            ├── FluidContainer (src/fluidcontainer.js)
            │
            ├── Readable (src/readable.js)
            │
            ├── Door (src/door.js)
            │
            ├── Key (src/key.js)
            │
            ├── Rune (src/rune.js)
            │
            └── Teleporter (src/teleporter.js)
```

---

## Thing (Base Class)

**File:** `src/thing.js`

Thing is the base class for all objects in the game world.

### Constructor

```javascript
const Thing = function(id) {
  // Inherit from EventEmitter
  EventEmitter.call(this);
  
  // Server-side item identifier
  this.id = id;
  
  // Stack count (for stackable items like gold)
  this.count = 1;
  
  // Unique action identifier (for scripting)
  this.actionId = 0;
  
  // Decay timer reference
  this.duration = null;
}
```

### Prototype Lookup

Each thing has a prototype that defines its shared properties:

```javascript
// Get the prototype (shared data)
Thing.prototype.getPrototype = function() {
  return gameServer.database.getThingPrototype(this.id);
}

// Get the client-side sprite ID
Thing.prototype.getClientId = function() {
  return this.getPrototype().id;
}

// Get item name
Thing.prototype.getName = function() {
  return this.getPrototype().properties.name;
}

// Get item article ("a", "an", or nothing)
Thing.prototype.getArticle = function() {
  return this.getPrototype().properties.article;
}

// Get description
Thing.prototype.getDescription = function() {
  return this.getPrototype().properties.description;
}
```

### Flags

Things have flags that define their behavior:

```javascript
// Check a flag
thing.hasFlag(FLAG_PICKUPABLE);

// Common flags:
FLAG_BLOCK_SOLID      // Blocks movement
FLAG_BLOCK_PROJECTILE // Blocks projectiles
FLAG_PICKUPABLE       // Can be picked up
FLAG_MOVEABLE         // Can be moved
FLAG_STACKABLE        // Stacks (like gold)
FLAG_HANGABLE         // Can hang on walls
FLAG_HORIZONTAL       // Horizontal wall
FLAG_VERTICAL         // Vertical wall
FLAG_HAS_HEIGHT       // Has height (affects rendering)
```

### Type Checks

```javascript
thing.isStackable();      // Returns true if stackable
thing.isDecaying();       // Returns true if will decay
thing.isFluidContainer(); // Returns true if can hold fluids
thing.isContainer();      // Returns true if can hold items
thing.isSplash();         // Returns true if liquid splash
thing.isDistanceWeapon(); // Returns true if ranged weapon
```

### Count Management

```javascript
// Set the count (for stackables)
thing.setCount(50);

// Get current count
thing.getCount();  // or thing.count

// Add to count (clamped to max 100)
thing.addCount(10);

// Remove from count
thing.removeCount(5);
```

### Action ID

Action IDs allow scripting specific item instances:

```javascript
// Set action ID
thing.setActionId(1001);

// Check for action ID
thing.hasActionId();

// Get action ID
thing.getActionId();
```

### Weight

```javascript
// Get total weight (weight × count for stackables)
thing.getWeight();

// Set weight
thing.setWeight(15.00);
```

### Decay System

```javascript
// Check if item decays
thing.isDecaying();

// Schedule decay (called on creation)
thing.scheduleDecay();

// Decay callback (transforms item)
thing.__handleDecay = function() {
  // Transform to decay target
  let target = this.getPrototype().properties.decayTo;
  // Replace this item with target item
}
```

### Cleanup

```javascript
// Called when item is removed from world
thing.cleanup();

// Cancel any scheduled events
thing.cancelEvent();
```

---

## Item

**File:** `src/item.js`

Item extends Thing with item-specific functionality.

### Constructor

```javascript
const Item = function(id) {
  Thing.call(this, id);
}
```

### Stack Limit

```javascript
Item.prototype.MAXIMUM_STACK_COUNT = 100;
```

### Movement Flags

```javascript
// Can be picked up by players
item.isPickupable();

// Can be moved (but not necessarily picked up)
item.isMoveable();

// Blocks solid movement
item.isBlockSolid();

// Blocks projectiles
item.isBlockProjectile();
```

### Height

```javascript
// Some items add height (affects rendering depth)
item.hasHeight();
```

### Hangable System

```javascript
// Can be hung on walls
item.isHangable();

// Supports hangables (is a wall)
item.supportsHangable();

// Wall orientation
item.isHorizontal();
item.isVertical();
```

### Splitting Stacks

```javascript
// Split a stackable item
let splitItem = item.split(count);

// Example: Split 20 gold from stack of 50
let goldStack = gameServer.database.createThing(2148);
goldStack.setCount(50);

let portion = goldStack.split(20);
// goldStack now has 30
// portion is a new item with 20
```

### Serialization

```javascript
Item.prototype.toJSON = function() {
  return {
    "id": this.id,
    "count": this.count,
    "actionId": this.actionId,
    "duration": this.duration
  };
}
```

---

## Specialized Item Types

### FluidContainer

**File:** `src/fluidcontainer.js`

Holds liquids (vials, buckets, etc.)

```javascript
// Set the fluid type
fluidContainer.setFluidType(CONST.FLUID.WATER);

// Get fluid type
fluidContainer.getFluidType();

// Fluid types:
CONST.FLUID.WATER
CONST.FLUID.BLOOD
CONST.FLUID.SLIME
CONST.FLUID.MANA
CONST.FLUID.LIFE  // Health potion
```

### Readable

**File:** `src/readable.js`

Books, scrolls, signs with text content.

```javascript
// Set content
readable.setContent("Hello, world!");

// Get content
readable.getContent();

// Check if readable at distance
readable.isDistanceReadable();
```

### Door

**File:** `src/door.js`

Doors with open/close states and locks.

```javascript
// Open the door
door.open();

// Close the door
door.close();

// Check state
door.isOpen();
door.isClosed();

// Check if locked
door.isLocked();
```

### Key

**File:** `src/key.js`

Keys for locked doors.

```javascript
// Set action ID (must match door)
key.setActionId(doorActionId);

// Check if matches
key.matches(door);
```

### Rune

**File:** `src/rune.js`

Magic runes with charges.

```javascript
// Get charges remaining
rune.getCharges();

// Use a charge
rune.useCharge();

// Check if depleted
rune.isDepleted();
```

### Teleporter

**File:** `src/teleporter.js`

Teleportation items/tiles.

```javascript
// Set destination
teleporter.setDestination(position);

// Get destination
teleporter.getDestination();
```

---

## Containers

**File:** `src/container.js`

Containers hold other items (backpacks, chests, etc.)

### Constructor

```javascript
const Container = function(id, size) {
  Item.call(this, id);
  
  // Number of slots
  this.size = size;
  
  // Unique identifier for network
  this.guid = Container.prototype.guid++;
  
  // Array of items (null = empty slot)
  this.__slots = new Array(size).fill(null);
  
  // Players viewing this container
  this.__spectators = new Set();
}
```

### Slot Management

```javascript
// Get item at index
container.getItemByIndex(index);

// Get all slots
container.getSlots();

// Get count of items
container.getItemCount();

// Check if empty
container.isEmpty();

// Check if full
container.isFull();

// Check if has item with ID
container.hasItem(itemId);

// Count items with ID
container.countItem(itemId);
```

### Adding Items

```javascript
// Add item at specific index
container.addThing(item, index);

// Add to first available slot
container.pushItem(item);

// Check if can add
container.canPushItem(item);
```

### Removing Items

```javascript
// Remove item at index
container.removeThing(index);

// Remove specific count
container.removeIndex(index, count);

// Remove all of a type
container.removeItemType(itemId, count);
```

### Spectators

Players viewing the container:

```javascript
// Add viewer
container.addSpectator(player);

// Remove viewer
container.removeSpectator(player);

// Notify all viewers of changes
container.notifyChange();
```

### Serialization

```javascript
Container.prototype.toJSON = function() {
  return {
    "id": this.id,
    "items": this.__slots.map(item => item ? item.toJSON() : null)
  };
}
```

### Corpse

**File:** `src/corpse.js`

Corpses are containers that decay.

```javascript
const Corpse = function(id, size) {
  Container.call(this, id, size);
  
  // Schedule decay to bones
  this.scheduleDecay();
}
```

---

## Equipment System

**File:** `src/equipment.js`

Equipment manages the player's worn items.

### Equipment Slots

```javascript
const EQUIPMENT_SLOTS = {
  HEAD: 0,      // Helmet
  ARMOR: 1,     // Armor
  LEGS: 2,      // Leg armor
  BOOTS: 3,     // Boots
  RIGHT: 4,     // Right hand (weapon)
  LEFT: 5,      // Left hand (shield)
  BACKPACK: 6,  // Backpack
  SHOULDER: 7,  // Amulet
  RING: 8,      // Ring
  QUIVER: 9     // Ammunition
};
```

### Equipment Class

```javascript
const Equipment = function(player, slots) {
  Container.call(this, 0, 10);  // 10 slots
  
  this.player = player;
  
  // Load saved equipment
  if(slots) {
    this.loadSlots(slots);
  }
}
```

### Slot Management

```javascript
// Get item in slot
equipment.getSlot(CONST.EQUIPMENT.HEAD);

// Equip item
equipment.equip(item, slotIndex);

// Unequip item
equipment.unequip(slotIndex);

// Check if slot is empty
equipment.isSlotEmpty(slotIndex);
```

### Attribute Calculation

```javascript
// Get total armor value
let armor = equipment.getAttributeState("armor");

// Get total attack value
let attack = equipment.getAttributeState("attack");

// Check weapon type
let weaponType = equipment.getWeaponType();
// Returns: CONST.PROPERTIES.SWORD, CLUB, AXE, DISTANCE, FIST
```

### Ammunition

```javascript
// Check if has distance weapon
equipment.isDistanceWeaponEquipped();

// Check if has ammunition
equipment.isAmmunitionEquipped();

// Consume one ammunition
equipment.removeIndex(CONST.EQUIPMENT.QUIVER, 1);
```

### Currency Operations

```javascript
// Pay with gold
equipment.payWithResource(2148, price);  // 2148 = gold coin ID

// Count gold available
equipment.countResource(2148);
```

---

## Item Stacks on Tiles

**File:** `src/item-stack.js`

Tiles use ItemStack to manage multiple items.

### Structure

```javascript
const ItemStack = function() {
  this.__items = [];  // Bottom to top order
}
```

### Adding Items

```javascript
// Add to top of stack
itemStack.addTopThing(item);

// Add at specific index
itemStack.addThing(item, index);

// Note: Ground items are index 0, others above
```

### Removing Items

```javascript
// Remove from index
itemStack.removeThing(index);

// Remove and return item
let item = itemStack.removeThing(index);
```

### Querying

```javascript
// Get item at index
itemStack.getThing(index);

// Get top item
itemStack.getTopThing();

// Get count
itemStack.getCount();

// Check if blocks solid
itemStack.isBlockSolid();

// Check if blocks projectile
itemStack.isBlockProjectile();
```

### Broadcasting Changes

```javascript
// When item added
tile.broadcast(new ItemAddPacket(position, item, index));

// When item removed
tile.broadcast(new ItemRemovePacket(position, index, count));
```

---

## Container Manager

**File:** `src/container-manager.js`

Manages all of a player's containers.

### Structure

```javascript
const ContainerManager = function(player, data) {
  this.player = player;
  
  // Equipment container
  this.equipment = new Equipment(player, data.equipment);
  
  // Keyring (for keys)
  this.keyring = new Container(2853, 20);
  
  // Depot (bank storage)
  this.depot = new Depot(player);
  
  // Open containers
  this.__openContainers = new Map();
}
```

### Opening Containers

```javascript
// Open a container
containerManager.openContainer(container);

// Close a container
containerManager.closeContainer(container);

// Close all containers
containerManager.closeAll();

// Get open container by ID
containerManager.getContainer(guid);
```

### Cleanup

```javascript
// Called on logout
containerManager.cleanup();
// Closes all open containers
```

---

## Item Creation Flow

```javascript
// 1. Get item from database
let item = gameServer.database.createThing(itemId);

// 2. Configure item
item.setCount(10);  // For stackables
item.setActionId(1001);  // For scripted items

// 3. Add to world
world.addTopThing(position, item);

// OR add to container
container.pushItem(item);

// OR add to equipment
player.containerManager.equipment.equip(item, slotIndex);
```

---

## Summary

| Class | Purpose | Stackable | Contains Items |
|-------|---------|-----------|----------------|
| **Thing** | Base class | Via flag | No |
| **Item** | Generic items | Via flag | No |
| **Container** | Holds items | No | Yes |
| **Corpse** | Dead creature | No | Yes (loot) |
| **FluidContainer** | Holds liquids | No | No |
| **Readable** | Has text | No | No |
| **Door** | Opens/closes | No | No |
| **Key** | Unlocks doors | No | No |
| **Rune** | Has charges | Yes (charges) | No |
| **Teleporter** | Teleports | No | No |

### Key Concepts

1. **Prototypes** - Shared item data from JSON definitions
2. **Instances** - Individual items with their own state
3. **Flags** - Boolean properties like pickupable, stackable
4. **Containers** - Can hold other items
5. **Equipment** - Special 10-slot container for worn items
6. **Action IDs** - Unique identifiers for scripted items

---

*Previous: [Entities](./04-ENTITIES.md) | Next: [Networking →](./06-NETWORKING.md)*

