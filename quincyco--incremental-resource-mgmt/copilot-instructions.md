## incremental-resource-mgmt

> **Project Name:** Incremental Resource Mgmt

# CLAUDE.md - Incremental Resource Management Game

## Project Overview

**Project Name:** Incremental Resource Mgmt
**Engine:** Godot 4.5
**Language:** GDScript
**Genre:** Incremental/Idle Game
**Current Status:** Early development with core mechanics implemented

This is an incremental game where players click on a rock/wizard to spawn particles, which can be collected for energy. Energy is used to purchase upgrades that enhance particle generation and collection efficiency.

---

## Codebase Structure

```
/
├── .git/                    # Git repository
├── .gitignore              # Godot-specific ignores
├── README.md               # Minimal project description
├── CLAUDE.md               # This file - AI assistant guide
├── project.godot           # Godot project configuration
│
├── main.tscn/.gd           # Root scene - connects all systems
├── game_manager.tscn/.gd   # Core game state and logic
├── ui_manager.tscn/.gd     # UI rendering and upgrade buttons
├── wizard.tscn/.gd         # Clickable object that spawns particles
├── particle.tscn/.gd       # Individual particle entities
├── particle_collector.tscn/.gd  # Collection area for particles
├── camera_controller.tscn/.gd   # Camera pan/zoom controls
└── icon.svg                # Godot default icon
```

---

## Architecture & Design Patterns

### Node Hierarchy

```
Main (Node2D)
├── GameManager (Node)           # Singleton-like game state
├── CameraController (Camera2D)  # Camera controls
├── UIManager (CanvasLayer)      # UI overlay
├── Wizard (Node2D)              # Click source
└── ParticleCollector (Node2D)   # Collection area
```

### Signal-Based Communication

The project uses Godot's signal system for decoupled communication:

**main.gd:10-11** - Signal connections:
```gdscript
game_manager.energy_changed.connect(ui_manager.update_energy_display)
ui_manager.upgrade_purchased.connect(game_manager.apply_upgrade)
```

**Key Signals:**
- `GameManager.energy_changed(new_energy: int)` - Emitted when energy updates
- `GameManager.particle_collected(value: int)` - Emitted on particle collection
- `GameManager.upgrade_applied(upgrade_id: String)` - Emitted after upgrade purchase
- `UIManager.upgrade_purchased(upgrade_id: String)` - Request upgrade purchase
- `Particle.particle_clicked(particle: RigidBody2D)` - Particle click event
- `Wizard.rock_clicked(click_position: Vector2)` - Rock/wizard click event

### Singleton Pattern (Godot Style)

GameManager is accessed as a pseudo-singleton via absolute node paths:
```gdscript
game_manager = get_node("/root/Main/GameManager")
```

This pattern is used in:
- particle.gd:19
- wizard.gd:13
- ui_manager.gd:13
- particle_collector.gd:12

---

## Game Mechanics

### Core Loop

1. Player clicks the Wizard (rock sprite)
2. Wizard spawns N particles (based on upgrades) with random velocities
3. Particles become RigidBody2D objects with physics
4. Player clicks particles OR they enter the collector area
5. Particles move toward the collector
6. Upon collection, energy is added to the player's total
7. Energy is spent on upgrades to enhance the loop

### Energy System

**Initial State:**
- Starting energy: 0
- Particles per click: 5
- Particle value: 1 energy each
- Collection radius: 50 units

**game_manager.gd:9-13** defines the initial game state.

### Upgrade System

**Upgrade Types** (game_manager.gd:16-21):

| Upgrade ID | Effect | Base Cost | Cost Scaling |
|-----------|--------|-----------|--------------|
| `more_particles` | +2 particles per click | 50 | 1.5x per level |
| `particle_value` | +1 energy per particle | 100 | 1.5x per level |
| `collection_radius` | +25 units collection area | 75 | 1.5x per level |
| `auto_collector` | +1 automatic collector | 200 | 1.5x per level |

**Cost Formula:** `new_cost = current_cost * 1.5` (game_manager.gd:48)

### Particle Lifecycle

**particle.gd** implements the full particle lifecycle:

1. **Spawn** - Created by wizard.gd:69 with random position/velocity
2. **Physics Phase** - Falls with gravity (0.5x), affected by linear_damp (2.0)
3. **Click Detection** - Area2D detects mouse clicks (particle.gd:59-63)
4. **Collection Phase** - Moves toward collector at increasing speed (particle.gd:50-57)
5. **Collection** - Adds value to energy, plays fade animation (particle.gd:91-100)
6. **Timeout** - Disappears after 10 seconds if uncollected (particle.gd:102-108)

**Critical Fix Notes:**
- particle.gd:17 - Particles must add themselves to "particles" group in code
- wizard.gd:59 - Particle scene path uses lowercase "particle.tscn"

---

## Development Workflow

### Running the Project

```bash
# Open in Godot Editor
godot project.godot

# Run from command line (if Godot is in PATH)
godot --path . --headless  # For testing
```

### Git Workflow

**Current Branch:** `claude/claude-md-mi3saow3iof548o1-01Q4SvRNiHggmSr4AmD39J6y`
**Main Branch:** (not specified - likely `main` or `master`)

**Branch Naming:** All development branches should start with `claude/` for AI assistant work.

### Scene Editing

- Scenes (.tscn) are in text format for version control
- Each .gd script has a corresponding .tscn file
- Scripts are attached to scene root nodes
- UI is built programmatically in ui_manager.gd:21-51 (no .tscn UI elements)

---

## Key Conventions & Best Practices

### Code Style

1. **Snake_case** for variables and functions
2. **PascalCase** for class/node names
3. **ALL_CAPS** for constants (not used extensively yet)
4. **Type hints** used consistently: `var total_energy: int = 0`
5. **Export variables** for scene references: `@export var particle_scene: PackedScene`

### Node References

**Preferred Pattern:**
```gdscript
@onready var sprite = $Sprite2D  # Relative path for owned nodes
var game_manager = get_node("/root/Main/GameManager")  # Absolute for singletons
```

### Debug Printing

Extensive debug prints are used throughout the codebase:
- particle.gd:34, 62, 76, 92, 104
- wizard.gd:39, 66
- game_manager.gd:27, 31

**Convention:** Keep debug prints during development, comment out for release.

### Resource Loading

**Dynamic loading pattern** (wizard.gd:58-59):
```gdscript
if not particle_scene:
    particle_scene = load("res://particle.tscn")
```

Always use lowercase filenames matching the actual file on disk.

### Placeholder Graphics

When no texture is assigned, scripts generate placeholder graphics programmatically:
- wizard.gd:20-32 - Gray rock circle
- particle.gd:36-48 - Blue glowing particle
- particle_collector.gd:26-47 - Brown bucket shape

**Pattern:** Always implement `_create_placeholder_texture()` for visual nodes.

---

## Known Issues & TODOs

### High Priority

1. **Save/Load System** (game_manager.gd:80-94)
   - Currently stubbed out with print statements
   - Need to implement file I/O with JSON or Godot's ConfigFile
   - Should save: energy, upgrade levels, all game state

2. **Auto Collectors Not Implemented**
   - Upgrade exists (auto_collector) but has no effect
   - Need timer system to generate passive energy
   - Should reference collector count: game_manager.gd:13

3. **Performance Optimization** (ui_manager.gd:127-129)
   - `_process()` updates button states every frame
   - Should only update on energy_changed signal

### Medium Priority

4. **Animation System**
   - wizard.gd:8 references AnimationPlayer that doesn't exist in .tscn
   - Falls back to tween animation (wizard.gd:51-54)
   - Consider adding proper hit animations

5. **Scene Organization**
   - All files in root directory
   - Should organize into folders: `scenes/`, `scripts/`, `assets/`

6. **Input Handling**
   - project.godot defines WASD/Arrow keys but only used for camera
   - Could add keyboard shortcuts for upgrades

### Low Priority

7. **Magic Numbers**
   - Many hardcoded values: particle lifetime (10s), impulse ranges, etc.
   - Consider moving to exported variables or configuration file

8. **Error Handling**
   - Limited null checks when accessing nodes
   - particle.gd:78 checks for null collector, pattern should be expanded

---

## File Reference Guide

### game_manager.gd (95 lines)

**Purpose:** Central game state manager
**Key Functions:**
- `add_energy(amount: int)` - Increments total energy, emits signals
- `apply_upgrade(upgrade_id: String) -> bool` - Purchases and applies upgrades
- `can_afford_upgrade(upgrade_id: String) -> bool` - Checks affordability
- `save_game_state()` / `load_game_state()` - Persistence (TODO)

**Important Variables:**
- `total_energy: int` - Player's currency
- `particles_per_click: int` - Number spawned per wizard click
- `particle_value: int` - Energy per particle collected
- `collection_radius: float` - Size of collector area
- `auto_collectors: int` - Count of passive collectors (unused)
- `upgrades: Dictionary` - Upgrade definitions with cost/level

### main.gd (14 lines)

**Purpose:** Root scene initialization
**Responsibilities:**
- Connects GameManager signals to UIManager
- Connects UIManager signals to GameManager
- Entry point for the game

**Critical:** Signal connections must happen in _ready() before any game events.

### wizard.gd (82 lines)

**Purpose:** Clickable object that spawns particles
**Key Functions:**
- `_on_rock_clicked()` - Handles click events
- `spawn_particles()` - Instantiates particle scenes with physics
- `_play_simple_bounce()` - Visual feedback animation

**Physics:** Particles spawn with random angle/distance and random impulse velocity.

### particle.gd (109 lines)

**Purpose:** Individual collectable particle entity
**Key Functions:**
- `start_moving_to_collector()` - Initiates collection sequence
- `collect()` - Awards energy and destroys particle
- `apply_initial_impulse(impulse: Vector2)` - Initial physics force

**State Machine:**
- Free-falling with gravity → Clicked/Entered collector → Moving to collector → Collected
- Alternative: Lifetime timeout → Fade out → Destroyed

**Physics Properties:**
- `gravity_scale: 0.5` - Half normal gravity
- `linear_damp: 2.0` - Air resistance
- `move_speed: 400.0` - Collection movement speed (accelerates)

### particle_collector.gd (65 lines)

**Purpose:** Collection area that attracts particles
**Key Functions:**
- `_on_body_entered(body: Node2D)` - Detects particles entering area
- `_update_collection_radius()` - Adjusts size based on upgrades

**Position:** Fixed at scene position, particles reference it via absolute path.

### ui_manager.gd (130 lines)

**Purpose:** User interface rendering and interaction
**Key Functions:**
- `update_energy_display(new_energy: int)` - Updates energy label
- `_create_upgrade_buttons()` - Generates upgrade UI programmatically
- `_on_upgrade_button_pressed(upgrade_id: String, button: Button)` - Purchase handler

**UI Structure:**
- HUD (top-left) - Energy display
- UpgradesPanel (left side) - Scrollable upgrade buttons

**Note:** All UI created in code, not in .tscn file.

### camera_controller.gd (74 lines)

**Purpose:** Camera pan and zoom controls
**Input:**
- **Mouse Wheel:** Zoom in/out
- **Middle Mouse Drag:** Pan camera
- **WASD/Arrows:** Keyboard pan

**Key Functions:**
- `focus_on_position(target_position: Vector2, duration: float)` - Smooth camera movement
- `reset_camera()` - Returns to origin with zoom 1.0

**Exports:**
- `pan_speed: 500.0`
- `zoom_speed: 0.1`
- `min_zoom: 0.3`, `max_zoom: 2.0`

---

## Common Tasks for AI Assistants

### Adding a New Upgrade

1. Add entry to `game_manager.gd:16-21` upgrades dictionary
2. Add effect case in `game_manager.gd:51-63` match statement
3. Add button definition in `ui_manager.gd:54-59` upgrade_definitions
4. Update `ui_manager.gd:118` upgrade_ids array for button mapping
5. Test affordability and cost scaling

### Implementing Save/Load

1. Implement `save_game_state()` in game_manager.gd:80-90
   - Use `FileAccess.open("user://savegame.json", FileAccess.WRITE)`
   - Serialize save_data dictionary to JSON
2. Implement `load_game_state()` in game_manager.gd:92-94
   - Check if file exists
   - Parse JSON and apply to game state variables
3. Call save after each upgrade purchase (already done at game_manager.gd:67)
4. Consider auto-save timer in GameManager._process()

### Adding New Particle Types

1. Create new scene extending particle.tscn
2. Override `value` variable or add multipliers
3. Modify `wizard.spawn_particles()` to randomly select particle types
4. Add visual distinction (color, texture, size)

### Implementing Auto Collectors

1. Add `_process(delta)` to game_manager.gd
2. Track time since last auto-collection
3. Every N seconds, call `add_energy(auto_collectors * particle_value)`
4. Add UI indicator showing passive income rate
5. Consider upgrade to increase auto-collector speed

### Reorganizing File Structure

**Proposed structure:**
```
/
├── project.godot
├── README.md
├── CLAUDE.md
├── scenes/
│   ├── main.tscn
│   ├── game_manager.tscn
│   ├── ui/
│   │   └── ui_manager.tscn
│   ├── entities/
│   │   ├── wizard.tscn
│   │   ├── particle.tscn
│   │   └── particle_collector.tscn
│   └── camera/
│       └── camera_controller.tscn
├── scripts/
│   ├── [corresponding .gd files]
└── assets/
    └── icon.svg
```

**Warning:** Moving files requires updating all absolute paths and may break scene references.

---

## Testing Guidelines

### Manual Testing Checklist

- [ ] Click wizard spawns correct number of particles
- [ ] Particles respond to clicks
- [ ] Particles auto-collect when entering collector area
- [ ] Energy counter updates correctly
- [ ] Each upgrade works as intended
- [ ] Upgrade costs increase correctly (1.5x)
- [ ] Upgrade buttons disable when unaffordable
- [ ] Particles disappear after 10 seconds if uncollected
- [ ] Camera pan and zoom work smoothly
- [ ] No errors in debug console during normal play

### Debug Console

Run game with `--verbose` flag to see all debug prints:
```bash
godot --path . --verbose
```

Key debug messages to watch for:
- "GameManager ready. Initial energy: 0" - Confirms initialization
- "Particle created at: ..." - Particle spawning
- "Particle clicked!" - Click detection
- "Particle collected! Value: N" - Collection success
- "ERROR: Could not find ParticleCollector!" - Critical path issue

---

## Performance Considerations

### Current Performance Characteristics

- **Particle Count:** Physics-based RigidBody2D particles can accumulate quickly
- **10-second lifetime** prevents infinite particle accumulation
- **Update Frequency:** UI updates every frame (optimization needed)

### Optimization Opportunities

1. **Object Pooling:** Reuse particle instances instead of queue_free()
2. **Batch Signals:** Collect multiple particles before emitting energy_changed
3. **Conditional UI Updates:** Only update buttons on signal, not every frame
4. **Particle Culling:** Despawn particles outside camera view
5. **Static Bodies:** Use StaticBody2D for collected particles instead of physics

---

## Extension Ideas

### Gameplay Enhancements

- Multiple wizard types with different particle patterns
- Particle combos for clicking multiple particles quickly
- Prestige system for long-term progression
- Achievement system
- Visual effects for upgrades and milestones

### Technical Improvements

- Particle system using GPUParticles2D for visual effects
- Sound effects and background music
- Settings menu (volume, quality, keybinds)
- Tutorial/help system
- Statistics tracking (total clicks, particles collected, etc.)

### Multiplayer/Social

- Leaderboards for energy totals
- Shared particle pools
- Competitive clicking events

---

## Glossary

- **GDScript:** Python-like scripting language for Godot
- **Node:** Base unit in Godot's scene tree
- **Scene:** Reusable collection of nodes saved as .tscn
- **Signal:** Event system for node communication
- **@onready:** Deferred initialization after node enters tree
- **@export:** Variable exposed to Godot editor
- **RigidBody2D:** Physics-enabled 2D object
- **Area2D:** Detection zone for overlaps/collisions
- **CanvasLayer:** UI overlay rendering layer
- **Tween:** Animation system for property interpolation

---

## Version History

- **Current State (Nov 2025):** Core mechanics functional, placeholders for graphics
- **Recent Commits:**
  - d557310 - "Updated code to be usable"
  - 9fb5419 - "Initial Code"
  - 3a1db96 - "Initial commit"

---

## Resources

- [Godot Documentation](https://docs.godotengine.org/en/stable/)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [Godot Incremental Game Tutorial](https://docs.godotengine.org/en/stable/community/tutorials.html)

---

**Last Updated:** 2025-11-17
**Maintained By:** Claude AI Assistant
**For Questions:** Review code comments and debug prints for implementation details

---
> Source: [quincyco/Incremental_Resource_Mgmt](https://github.com/quincyco/Incremental_Resource_Mgmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
