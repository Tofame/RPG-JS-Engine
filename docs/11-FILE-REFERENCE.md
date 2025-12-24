# Complete File Reference

This document provides a reference for every source file in the project.

---

## Server Source Files (`src/`)

### Core Server

| File | Description |
|------|-------------|
| `gameserver.js` | Main server class, orchestrates all components |
| `gameloop.js` | 50ms tick timer for game simulation |
| `database.js` | Loads and manages all game data |
| `database-action-loader.js` | Loads action scripts and attaches events |

### World & Spatial

| File | Description |
|------|-------------|
| `world.js` | Game world container, channels, events |
| `lattice.js` | 2D grid of chunks for spatial organization |
| `chunk.js` | Section of world, contains tiles and creatures |
| `tile.js` | Single map tile with items and properties |
| `position.js` | 3D position class (x, y, z) |
| `geometry.js` | Geometric calculations (distance, direction) |
| `pathfinder.js` | A* pathfinding algorithm |
| `pathfinder-node.js` | Node class for pathfinding |

### Entities

| File | Description |
|------|-------------|
| `creature.js` | Base class for all living entities |
| `creature-properties.js` | Property management for creatures |
| `player.js` | Player entity with all handlers |
| `player-properties.js` | Player-specific properties |
| `player-skills.js` | Skill system and experience |
| `player-movement-handler.js` | Walking and teleporting |
| `player-action-handler.js` | Use, look, attack actions |
| `player-use-handler.js` | Item usage logic |
| `player-socket-handler.js` | Network communication |
| `player-idle-handler.js` | AFK timeout management |
| `player-combat-lock.js` | Combat state tracking |
| `player-channel-manager.js` | Chat channel membership |
| `monster.js` | Monster entity with AI |
| `monster-behaviour.js` | Monster AI (chase, flee, attack) |
| `monster-loot-handler.js` | Loot generation on death |
| `monster-loot-entry.js` | Single loot table entry |
| `npc.js` | NPC entity with conversation |
| `npc-behaviour-handler.js` | NPC movement behavior |
| `npc-conversation-handler.js` | Dialog system |
| `npc-talk-state-handler.js` | Conversation state machine |
| `npc-trade-handler.js` | Shop/trading system |
| `npc-focus-handler.js` | NPC attention management |
| `npc-scene-handler.js` | Scripted sequences |
| `npc-scene-action.js` | Single scene action |

### Items & Containers

| File | Description |
|------|-------------|
| `thing.js` | Base class for all objects |
| `thing-prototype.js` | Shared item properties |
| `thing-emitter.js` | Event emission for things |
| `item.js` | Generic item class |
| `item-stack.js` | Stack of items on a tile |
| `container.js` | Item that holds other items |
| `base-container.js` | Base container functionality |
| `container-manager.js` | Player's containers |
| `equipment.js` | Player's worn items |
| `corpse.js` | Dead creature container |
| `fluidcontainer.js` | Liquid-holding items |
| `readable.js` | Books, scrolls, signs |
| `door.js` | Door with open/close states |
| `key.js` | Keys for locked doors |
| `keyring.js` | Key storage container |
| `rune.js` | Magic runes with charges |
| `teleporter.js` | Teleportation items |
| `depot.js` | Player bank storage |
| `house.js` | House ownership |

### Combat & Damage

| File | Description |
|------|-------------|
| `world-combat-handler.js` | Combat resolution |
| `damage-map.js` | Tracks damage sources |
| `damage-map-entry.js` | Single damage source entry |
| `target-handler.js` | Target selection |

### Conditions & Skills

| File | Description |
|------|-------------|
| `condition.js` | Status effect class |
| `condition-manager.js` | Active conditions on creature |
| `skill.js` | Single skill definition |
| `skills.js` | All player skills |
| `spellbook.js` | Learned spells |

### Networking

| File | Description |
|------|-------------|
| `http-server.js` | HTTP server wrapper |
| `websocket-server.js` | WebSocket server |
| `websocket-server-socket-handler.js` | Connection management |
| `gamesocket.js` | Single client connection |
| `network-manager.js` | Network utilities |
| `bandwidth-handler.js` | Traffic monitoring |
| `packet.js` | Base packet class |
| `packet-buffer.js` | Packet buffer utilities |
| `packet-reader.js` | Parse incoming packets |
| `packet-writer.js` | Create outgoing packets |
| `packet-handler.js` | Route packets to handlers |
| `protocol.js` | All packet definitions |

### Channels & Communication

| File | Description |
|------|-------------|
| `channel.js` | Base chat channel |
| `channel-manager.js` | All server channels |
| `channel-default.js` | Local say channel |
| `channel-global.js` | World/trade channels |
| `speech-handler.js` | Creature speech |
| `friendlist.js` | Player friend list |
| `inbox.js` | Player mail system |
| `mailbox-handler.js` | Mail processing |

### Events & Actions

| File | Description |
|------|-------------|
| `eventqueue.js` | Scheduled events |
| `binary-heap.js` | Priority queue implementation |
| `event.js` | Base event class |
| `eventemitter.js` | Event emission pattern |
| `actions.js` | Action cooldown system |
| `command-handler.js` | Slash commands |

### World Features

| File | Description |
|------|-------------|
| `world-clock.js` | Day/night cycle |
| `world-creature-handler.js` | Creature management |
| `outfit.js` | Creature appearance |

### Map Parsing

| File | Description |
|------|-------------|
| `otbm-parser.js` | OTBM map file parser |
| `otbm-packet-reader.js` | OTBM binary reader |
| `otbm-node.js` | OTBM node structure |
| `otbm-headers.js` | OTBM format constants |

### Authentication

| File | Description |
|------|-------------|
| `login-server.js` | Login server |
| `account-database.js` | SQLite account storage |
| `auth-service.js` | Authentication utilities |
| `character-creator.js` | New character creation |

### IPC

| File | Description |
|------|-------------|
| `ipcsocket.js` | Inter-process communication |
| `ipcclient.js` | IPC client |
| `ipcpacket.js` | IPC packet format |
| `ipchttpapi.js` | IPC HTTP API |

### Utilities

| File | Description |
|------|-------------|
| `__proto__.js` | Prototype extensions |
| `enum.js` | Enumeration helper |
| `bitflag.js` | Bitflag operations |
| `property.js` | Property class |
| `validator.js` | JSON schema validation |
| `logger.js` | Logging utilities |
| `generic-lock.js` | Cooldown locks |

---

## Client Source Files (`client/src/`)

### Core Client

| File | Description |
|------|-------------|
| `index.js` | Entry point |
| `gameclient.js` | Main client class |
| `game-loop.js` | Animation frame loop |
| `database.js` | Sprite/object storage |
| `settings.js` | User settings |
| `state.js` | Client state management |

### Rendering

| File | Description |
|------|-------------|
| `canvas.js` | Main canvas wrapper |
| `renderer.js` | Frame rendering |
| `sprite.js` | Sprite class |
| `sprite-buffer.js` | Sprite caching |
| `object-buffer.js` | Object caching |
| `light-canvas.js` | Lighting overlay |
| `weather-canvas.js` | Weather effects |
| `outline-canvas.js` | Selection outlines |
| `animation.js` | Animation system |
| `distance-animation.js` | Projectile effects |
| `box-animation.js` | Area effects |
| `frame-group.js` | Animation frames |

### World State

| File | Description |
|------|-------------|
| `world.js` | Client world state |
| `chunk.js` | Map chunk |
| `tile.js` | Map tile |
| `position.js` | Position class |
| `pathfinder.js` | Client pathfinding |
| `minimap.js` | Minimap rendering |
| `clock.js` | World clock display |

### Entities

| File | Description |
|------|-------------|
| `creature.js` | Base creature |
| `player.js` | Local player |
| `monster.js` | Monster display |
| `outfit.js` | Outfit rendering |
| `condition.js` | Status effects |
| `skills.js` | Skill display |
| `spellbook.js` | Spell list |

### Items

| File | Description |
|------|-------------|
| `thing.js` | Base object |
| `item.js` | Item class |
| `fluid-container.js` | Fluid display |
| `book.js` | Readable content |
| `dataobject.js` | Object data |

### Networking

| File | Description |
|------|-------------|
| `network-manager.js` | WebSocket client |
| `packet.js` | Base packet |
| `packetreader.js` | Read packets |
| `packetwriter.js` | Write packets |
| `packet-handler.js` | Handle packets |
| `protocol.js` | Packet types |
| `replay-manager.js` | Replay recording |

### Interface

| File | Description |
|------|-------------|
| `interface.js` | UI management |
| `menu.js` | Base menu |
| `menu-manager.js` | Menu system |
| `menu-screen.js` | Screen context menu |
| `menu-hotbar.js` | Hotbar menu |
| `menu-message.js` | Message menu |
| `menu-chat-header.js` | Chat header menu |
| `menu-chat-body.js` | Chat body menu |
| `menu-friend-list.js` | Friend menu |
| `menu-friend-window.js` | Friend window menu |

### Windows

| File | Description |
|------|-------------|
| `window.js` | Base window |
| `window-manager.js` | Window system |
| `window-skill.js` | Skills window |
| `window-battle.js` | Battle list |
| `window-friend.js` | Friends list |
| `container.js` | Container window |
| `equipment.js` | Equipment panel |
| `slot.js` | Inventory slot |
| `status-bar.js` | Condition icons |
| `hotbar-manager.js` | F1-F12 hotbar |

### Modals

| File | Description |
|------|-------------|
| `modal.js` | Base modal |
| `modal-manager.js` | Modal system |
| `modal-confirm.js` | Confirmation |
| `modal-text.js` | Text display |
| `modal-enter-name.js` | Name input |
| `modal-move-item.js` | Item split |
| `modal-outfit.js` | Outfit change |
| `modal-chat.js` | Channel select |
| `modal-offer.js` | Trade window |
| `modal-readable.js` | Book/sign view |
| `modal-spellbook.js` | Spell list |
| `modal-map.js` | World map |
| `modal-create-account.js` | Registration |

### Chat & Channels

| File | Description |
|------|-------------|
| `channel.js` | Chat channel |
| `channel-manager.js` | Channel system |
| `private-channel.js` | Whisper channel |
| `local-channel.js` | Local console |
| `message.js` | Chat message |
| `message-character.js` | Character message |
| `friendlist.js` | Friend tracking |
| `casting-manager.js` | Spell casting |

### Screen Elements

| File | Description |
|------|-------------|
| `screen-element.js` | Base element |
| `screen-element-manager.js` | Element system |
| `screen-element-character.js` | Name/health bar |
| `screen-element-message.js` | Speech bubble |
| `screen-element-floating.js` | Floating text |
| `notification.js` | Toast messages |

### Input

| File | Description |
|------|-------------|
| `keyboard.js` | Keyboard handler |
| `mouse.js` | Mouse handler |

### Sound

| File | Description |
|------|-------------|
| `sound-manager.js` | Audio system |
| `soundbit.js` | Sound effect |
| `soundtrace.js` | Ambient music |

### Utilities

| File | Description |
|------|-------------|
| `__proto__.js` | Prototype extensions |
| `enum.js` | Enumeration helper |
| `bitflag.js` | Bitflag operations |
| `rgba.js` | Color utilities |
| `binary-heap.js` | Priority queue |
| `event-queue.js` | Event scheduling |
| `heap-event.js` | Queue event |
| `eventemitter.js` | Event pattern |
| `debugger.js` | Debug display |
| `error.js` | Error handling |

---

## Data Files (`data/740/`)

| Directory | Contents |
|-----------|----------|
| `items/` | Item definitions |
| `monsters/` | Monster definitions |
| `npcs/` | NPC definitions |
| `spells/` | Spell scripts |
| `conditions/` | Condition scripts |
| `actions/` | Item action scripts |
| `runes/` | Rune scripts |
| `doors/` | Door scripts |
| `unique/` | Unique item scripts |
| `clock/` | Time event scripts |
| `houses/` | House data |
| `spawns/` | Monster spawn points |
| `world/` | Map file (OTBM) |
| `outfits/` | Outfit definitions |
| `mounts/` | Mount definitions |
| `characters/` | Account templates |

---

*Previous: [Adding Features](./10-ADDING-FEATURES.md) | Back to [Index](./00-INDEX.md)*

