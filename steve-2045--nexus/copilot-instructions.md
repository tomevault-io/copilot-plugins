## nexus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Godot 4.4 game implementing a visual rock-paper-scissors simulation called "piedra_papel_tijera". The game features autonomous entities that move around the screen, collide with each other, and transform based on rock-paper-scissors rules.

## Architecture

### Core Game Logic
- **Main Scene**: `espacio/mundo.tscn` - Contains the game world with camera and entity instances
- **Base Character**: `personajes/base_character.gd` - Single script that handles all three emotions
  - Uses `emotion_type` enum to switch between "happy" (Rock), "angry" (Paper), and "sad" (Scissors)
  - Scene files: `angry.tscn`, `happy.tscn`, `sad.tscn` - All use the same BaseCharacter script

### Game Rules (Rock-Paper-Scissors)
- **Happy (Piedra/Rock)** beats **Sad (Tijera/Scissors)**
- **Sad (Tijera/Scissors)** beats **Angry (Papel/Paper)**
- **Angry (Papel/Paper)** beats **Happy (Piedra/Rock)**

### Entity Behavior System
The BaseCharacter class extends `RigidBody2D` and implements:
- Constant velocity movement with screen boundary collision
- Hunter-prey behavior system with speed modifiers
- Collision detection and emotion transformation logic
- Dynamic sprite texture switching based on current emotion
- Physics-based bouncing with velocity maintenance
- Area2D detection for hunting behavior

## Key Components

### BaseCharacter Class Structure
Single `base_character.gd` script with unified behavior:
- `emotion_type` - Enum identifier ("angry", "happy", "sad")
- `speed_min/speed_max` - Configurable velocity range (75-125)
- `is_hunting/is_being_hunted` - Hunter-prey state tracking
- `hunting_target` - Current prey reference
- `hunters[]` - Array of entities hunting this one
- `get_emotion_type()` - Returns current emotion type
- `change_to(new_type)` - Transforms entity to new emotion
- `can_hunt(other_type)` - Determines if can hunt given emotion type
- `process_collision(other_body)` - Handles game rule logic
- `maintain_constant_speed()` - Ensures consistent movement with hunting behavior
- `update_speed()` - Modifies speed based on hunting state (1.25x when hunting, 0.75x when hunted)

### Asset Organization
- `sprites/` - Contains emotion face textures (PNG format)
- `personajes/` - Character scripts and scene files (.tscn)
- `espacio/` - World/level scenes

## Development Commands

### Running the Game
```bash
# Run from Godot Editor or
godot --path . espacio/mundo.tscn
```

### Project Structure
- Main scene: `espacio/mundo.tscn`
- Project config: `project.godot`
- Assets: All resources use Godot's `res://` path system

## Technical Notes

- Uses Godot 4.4 with "Forward Plus" rendering
- Physics system: RigidBody2D with disabled gravity and rotation lock
- Collision system: Contact monitoring with up to 10 contacts per body
- Dual collision detection: RigidBody2D for contact and Area2D for hunting detection
- All entities maintain constant velocity (75-125 base units) with hunting speed modifiers
- Screen boundary detection uses viewport visible rect calculations
- Collision layer system: Each emotion type uses separate collision layers for selective interaction
- Hunter-prey mechanics: Entities actively pursue valid targets and flee from threats

---
> Source: [Steve-2045/nexus](https://github.com/Steve-2045/nexus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
