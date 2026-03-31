# Beyond World Mini-Game Editor — Concept Guide

This document explains the game editor's concepts for writing game logic specs.
You do NOT need to know how things are implemented — only what they are and how they interact.

---

## Entities

Everything in the game world is an **entity**. Types:

- **Character**: A playable character controlled by a player. Has HP, attack power, defense, movement speed, etc.
- **Creation / Object**: A placed object in the map — triggers, devices, decorations, monsters, NPCs.
- **Player**: A connected human. Owns one or more characters. Referenced by player ID (1-indexed).

Every entity has:
- A **GUID** (unique instance ID, integer)
- A **Prefab ID** (template type — "what kind of object is this?")
- A **Config ID** (configuration identifier)
- A **Faction** (team/side — factions can be hostile to each other)
- A **position** and **rotation** in 3D space (vec3)

Key references:
- `self` — the entity this logic belongs to
- `player(1)`, `player(2)` — player entities by ID
- `stage` / `level` — the game stage itself

---

## Events

Events fire automatically when something happens in the game. Your logic **reacts** to events.

Common events:
- Entity is created / destroyed
- Attack hits a target / entity is attacked
- Character goes down / revives
- Entity enters / exits a collision trigger
- Timer fires
- Signal is received
- Variable changes value
- Item is added to / removed from inventory
- Equipment is equipped / unequipped

Each event provides **output data** you can use:
- The entity that caused the event
- Relevant values (damage amount, HP, attacker, etc.)

---

## Signals

Signals are **messages** sent between different logic blocks.

- You define a signal by name (e.g., "BigHit", "WaveComplete")
- Signals can carry typed data (int, float, str, entity, etc.)
- One logic block sends → another receives and reacts
- Signals must be **pre-defined** in the editor with their argument types

Use signals to:
- Coordinate between separate logic blocks
- Trigger chain reactions
- Pass data between different parts of your game

---

## Variables

Three kinds of persistent state:

### Graph Variables
- Shared across the entire logic block
- Persist between events
- Declared upfront with a type and initial value
- Example: a kill counter, a damage accumulator

### Custom Variables
- Attached to a specific entity (key-value pairs)
- Each entity can have different values for the same key
- Example: individual HP override, per-entity score

### Local Variables
- Exist only during one event handler execution
- Temporary calculations, intermediate values
- Gone when the handler finishes

---

## Timers

Schedule delayed or repeated actions:

- **Delayed**: do something once after N milliseconds
- **Repeated**: do something every N milliseconds
- Timers can be paused, resumed, stopped
- Global timers persist across entity lifecycle

---

## Factions

Entities belong to factions (teams/sides).
- Factions can be set as hostile to each other
- Attacks only damage hostile faction entities (by default)
- You can query and change an entity's faction at runtime

---

## Collision Triggers

Invisible zones in the map that detect when entities enter or leave.
- Trigger events: "entity entered zone" / "entity exited zone"
- Query: "which entities are currently inside this zone?"

---

## Combat System

- **Attacks**: initiated by one entity against another
- **HP**: health points — recoverable, can be lost
- **Damage**: calculated from attack, reduced by defense
- **Shields**: absorb damage before HP
- **Elemental reactions**: triggered by combining elements
- **Down state**: when HP reaches 0 — can be revived

---

## Status Effects (Unit Status)

Buff/debuff system:
- Applied to entities with a config ID
- Have stack counts (can stack multiple times)
- Can be queried, added, removed
- Trigger events when applied/expired

---

## Preset Status

Simple integer values attached to entities:
- Used for custom state flags (alive/dead, phase 1/2/3, enabled/disabled)
- Set and read by config ID

---

## Shop & Inventory

- **Inventory**: per-player item/currency storage with capacity limits
- **Shops**: UI for buying/selling items, opened programmatically
- Items have config IDs and quantities
- Currency is separate from items

---

## Equipment

- Items that can be equipped to characters in slots
- Equipment has **affixes** (modifiers with values)
- Affixes can be added, removed, modified at runtime

---

## Loot

- Dropped items from defeated entities
- Loot contents can be configured programmatically
- Players pick up loot to add to inventory

---

## Other Systems

- **Mini-map markers**: show/hide points on the mini-map
- **Camera**: switch camera templates for cinematic effects
- **UI controls**: show/hide custom interface elements, tabs, text bubbles
- **Audio**: play/stop background music and sound effects
- **Tags**: label entities with unit tags for grouping/querying
- **Aggro**: AI targeting system (who attacks whom)
- **Patrol**: set entity movement patterns along waypoints
- **Achievement**: track and complete objectives
- **Settlement/Ranking**: end-of-game scoring and results
- **Deck Selector**: card/choice selection UI
- **Chat channels**: in-game communication
- **Environment**: time of day, weather settings
