## tamagocli

> Console-based Tamagotchi game with colorful ASCII art, multiple pet types, real-time stat decay, and smooth animations. Built with Python 3.10+, uses curses for flicker-free rendering, and uv for package management.


# Project Layout

## Overview

Console-based Tamagotchi game with colorful ASCII art, multiple pet types, real-time stat decay, and smooth animations. Built with Python 3.10+, uses curses for flicker-free rendering, and uv for package management.

## Architecture

The project follows a modular architecture with clear separation of concerns:

```
tamagochi/
├── tamagochi/          # Main package
│   ├── models/         # Game models and business logic
│   ├── display/        # Rendering and UI
│   ├── game/           # Game engine and save system
│   ├── utils/          # Utilities (menus, helpers)
│   └── main.py         # Entry point
├── tests/              # Test suite
├── pyproject.toml      # Project configuration
└── pytest.ini          # Test configuration
```

## Core Components

### 1. Models Layer (`tamagochi/models/`)

**`pet.py`** - Core game logic:
- `PetType` enum: CAT, DOG, DRAGON, BUNNY, ALIEN
- `PetState` enum: IDLE, HAPPY, HUNGRY, EATING, SLEEPING, SAD, SICK, DEAD
- `PetStats` dataclass: hunger, happiness, energy, health (0-100), age, level
- `Pet` class: Main game object with:
  - Type-specific traits (hunger_decay, happiness_decay, energy_decay, food_efficiency)
  - Real-time stat decay with smooth accumulation
  - Actions: feed(), play(), sleep(), wake_up(), heal()
  - Movement logic with boundary bouncing
  - State management based on stats
  - Serialization for save/load (to_dict/from_dict)

### 2. Display Layer (`tamagochi/display/`)

**`sprites.py`** - ASCII art sprites:
- Defines sprites for each PetType × PetState combination
- Multiple animation frames for idle states
- Returns sprite based on (pet_type, state, frame_index)

**`curses_renderer.py`** - Main game renderer:
- `CursesGameRenderer`: Flicker-free rendering with double buffering
- Differential rendering - only updates changed elements
- Color pairs for different states
- Renders: header, pet sprite, stats bars, controls, event log

**`renderer.py`** - Fallback renderer:
- `GameRenderer`: Rich-based renderer for non-curses environments
- Used in pet selection and messages

**`action_menu.py`** - Interactive action menu:
- `ActionMenu`: Arrow key navigation for game actions
- Actions: Feed (F), Play (P), Sleep (S), Heal (H), Quit (Q)

### 3. Game Layer (`tamagochi/game/`)

**`engine.py`** - Game loop:
- `GameEngine`: Main game loop with curses integration
- Non-blocking input (50ms timeout)
- Update cycle: 1 second for pet updates
- Auto-save every 30 seconds
- Handles keyboard input (arrow keys + letter commands)
- Event logging system

**`save_manager.py`** - Persistence:
- `SaveManager`: JSON-based save/load system
- Save location: `~/.tamagochi/save.json`
- Methods: save(), load(), has_save(), delete_save()

### 4. Utils Layer (`tamagochi/utils/`)

**`curses_menu.py`** - Menu system:
- `interactive_menu_curses()`: Arrow key menu with live preview
- `yes_no_menu_curses()`: Yes/No selection
- Two-column layout with preview panel
- Differential rendering for smooth performance

**`menu.py`** - Fallback menu utilities

### 5. Entry Point

**`main.py`** - Application entry:
- Terminal setup/restore
- Pet selection with preview
- Save game detection and loading
- Main game loop initialization
- Error handling and cleanup

## Key Design Patterns

### 1. **Differential Rendering**
- Tracks previous state (prev_pet_state, prev_stats, etc.)
- Only redraws changed elements
- Uses curses noutrefresh() + doupdate() for double buffering
- Result: Zero flickering, smooth animations

### 2. **Smooth Stat Decay**
- Accumulates fractional changes (accumulated_hunger_decay, etc.)
- Applies integer changes when accumulation >= 1.0
- Prevents rounding errors over long gameplay sessions
- Type-specific multipliers for balanced difficulty

### 3. **Eating Timer**
- Tracks eating_started_at timestamp
- Maintains EATING state for 3 seconds
- Blocks certain actions during eating
- Auto-transitions to IDLE after timer

### 4. **Event-Driven Updates**
- Pet.update() returns list of events
- Events displayed in scrolling log (last 10)
- Events triggered by state changes (starving, exhausted, etc.)

### 5. **Type Traits System**
- Each pet type has unique multipliers
- Affects: stat decay rates, food efficiency
- Provides distinct gameplay for each pet

## Data Flow

```
main() 
  → Pet Selection (curses_menu) 
  → Load/Create Pet (save_manager)
  → GameEngine.run()
    ┌─ Input Loop (curses, non-blocking)
    ├─ Pet.update() (every 1s)
    │   └─ Returns events
    ├─ CursesGameRenderer.render_game()
    │   └─ Differential updates only
    └─ SaveManager.save() (every 30s)
```

## Testing Structure

```
tests/
├── test_pet_model.py         # Pet creation, stats, actions, serialization
├── test_eating_timer.py      # 3-second eating timer logic
├── test_health_decay.py      # Health consequences
├── test_sleeping.py          # Sleep mechanics
├── test_stat_decay_realtime.py # Real-time decay simulation
└── test_sprite_alignment.py  # Sprite rendering tests
```

## Dependencies

**Core:**
- `rich>=13.7.0` - Rich text formatting (fallback renderer)
- `readchar>=4.0.0` - Character input (legacy, mostly replaced by curses)

**Dev:**
- `pytest>=7.4.0` - Testing framework
- `pytest-cov>=4.1.0` - Coverage reporting

**Built-in:**
- `curses` - Terminal UI (main renderer)
- `json` - Save file format
- `datetime` - Time tracking
- `dataclasses` - Data structures

## File Organization Principles

1. **Models** - Pure business logic, no UI dependencies
2. **Display** - UI code, depends on models
3. **Game** - Game loop and persistence, orchestrates models + display
4. **Utils** - Reusable components
5. **Tests** - Mirror source structure, focus on core logic

## Save File Format

JSON file at `~/.tamagochi/save.json`:
```json
{
  "name": "Fluffy",
  "pet_type": "cat",
  "stats": {
    "hunger": 85,
    "happiness": 90,
    "energy": 75,
    "health": 100,
    "age": 1234,
    "level": 1
  },
  "state": "happy",
  "position": 45,
  "direction": 1,
  "created_at": "2025-10-25T10:30:00",
  "last_update": "2025-10-25T10:35:00",
  "eating_started_at": null,
  "accumulated_damage": 0.0,
  "accumulated_hunger_decay": 0.5,
  "accumulated_happiness_decay": 0.3,
  "accumulated_energy_decay": 0.2
}
```
# Project Layout

## Overview

Console-based Tamagotchi game with colorful ASCII art, multiple pet types, real-time stat decay, and smooth animations. Built with Python 3.10+, uses curses for flicker-free rendering, and uv for package management.

## Architecture

The project follows a modular architecture with clear separation of concerns:

```
tamagochi/
├── tamagochi/          # Main package
│   ├── models/         # Game models and business logic
│   ├── display/        # Rendering and UI
│   ├── game/           # Game engine and save system
│   ├── utils/          # Utilities (menus, helpers)
│   └── main.py         # Entry point
├── tests/              # Test suite
├── pyproject.toml      # Project configuration
└── pytest.ini          # Test configuration
```

## Core Components

### 1. Models Layer (`tamagochi/models/`)

**`pet.py`** - Core game logic:
- `PetType` enum: CAT, DOG, DRAGON, BUNNY, ALIEN
- `PetState` enum: IDLE, HAPPY, HUNGRY, EATING, SLEEPING, SAD, SICK, DEAD
- `PetStats` dataclass: hunger, happiness, energy, health (0-100), age, level
- `Pet` class: Main game object with:
  - Type-specific traits (hunger_decay, happiness_decay, energy_decay, food_efficiency)
  - Real-time stat decay with smooth accumulation
  - Actions: feed(), play(), sleep(), wake_up(), heal()
  - Movement logic with boundary bouncing
  - State management based on stats
  - Serialization for save/load (to_dict/from_dict)

### 2. Display Layer (`tamagochi/display/`)

**`sprites.py`** - ASCII art sprites:
- Defines sprites for each PetType × PetState combination
- Multiple animation frames for idle states
- Returns sprite based on (pet_type, state, frame_index)

**`curses_renderer.py`** - Main game renderer:
- `CursesGameRenderer`: Flicker-free rendering with double buffering
- Differential rendering - only updates changed elements
- Color pairs for different states
- Renders: header, pet sprite, stats bars, controls, event log

**`renderer.py`** - Fallback renderer:
- `GameRenderer`: Rich-based renderer for non-curses environments
- Used in pet selection and messages

**`action_menu.py`** - Interactive action menu:
- `ActionMenu`: Arrow key navigation for game actions
- Actions: Feed (F), Play (P), Sleep (S), Heal (H), Quit (Q)

### 3. Game Layer (`tamagochi/game/`)

**`engine.py`** - Game loop:
- `GameEngine`: Main game loop with curses integration
- Non-blocking input (50ms timeout)
- Update cycle: 1 second for pet updates
- Auto-save every 30 seconds
- Handles keyboard input (arrow keys + letter commands)
- Event logging system

**`save_manager.py`** - Persistence:
- `SaveManager`: JSON-based save/load system
- Save location: `~/.tamagochi/save.json`
- Methods: save(), load(), has_save(), delete_save()

### 4. Utils Layer (`tamagochi/utils/`)

**`curses_menu.py`** - Menu system:
- `interactive_menu_curses()`: Arrow key menu with live preview
- `yes_no_menu_curses()`: Yes/No selection
- Two-column layout with preview panel
- Differential rendering for smooth performance

**`menu.py`** - Fallback menu utilities

### 5. Entry Point

**`main.py`** - Application entry:
- Terminal setup/restore
- Pet selection with preview
- Save game detection and loading
- Main game loop initialization
- Error handling and cleanup

## Key Design Patterns

### 1. **Differential Rendering**
- Tracks previous state (prev_pet_state, prev_stats, etc.)
- Only redraws changed elements
- Uses curses noutrefresh() + doupdate() for double buffering
- Result: Zero flickering, smooth animations

### 2. **Smooth Stat Decay**
- Accumulates fractional changes (accumulated_hunger_decay, etc.)
- Applies integer changes when accumulation >= 1.0
- Prevents rounding errors over long gameplay sessions
- Type-specific multipliers for balanced difficulty

### 3. **Eating Timer**
- Tracks eating_started_at timestamp
- Maintains EATING state for 3 seconds
- Blocks certain actions during eating
- Auto-transitions to IDLE after timer

### 4. **Event-Driven Updates**
- Pet.update() returns list of events
- Events displayed in scrolling log (last 10)
- Events triggered by state changes (starving, exhausted, etc.)

### 5. **Type Traits System**
- Each pet type has unique multipliers
- Affects: stat decay rates, food efficiency
- Provides distinct gameplay for each pet

## Data Flow

```
main() 
  → Pet Selection (curses_menu) 
  → Load/Create Pet (save_manager)
  → GameEngine.run()
    ┌─ Input Loop (curses, non-blocking)
    ├─ Pet.update() (every 1s)
    │   └─ Returns events
    ├─ CursesGameRenderer.render_game()
    │   └─ Differential updates only
    └─ SaveManager.save() (every 30s)
```

## Testing Structure

```
tests/
├── test_pet_model.py         # Pet creation, stats, actions, serialization
├── test_eating_timer.py      # 3-second eating timer logic
├── test_health_decay.py      # Health consequences
├── test_sleeping.py          # Sleep mechanics
├── test_stat_decay_realtime.py # Real-time decay simulation
└── test_sprite_alignment.py  # Sprite rendering tests
```

## Dependencies

**Core:**
- `rich>=13.7.0` - Rich text formatting (fallback renderer)
- `readchar>=4.0.0` - Character input (legacy, mostly replaced by curses)

**Dev:**
- `pytest>=7.4.0` - Testing framework
- `pytest-cov>=4.1.0` - Coverage reporting

**Built-in:**
- `curses` - Terminal UI (main renderer)
- `json` - Save file format
- `datetime` - Time tracking
- `dataclasses` - Data structures

## File Organization Principles

1. **Models** - Pure business logic, no UI dependencies
2. **Display** - UI code, depends on models
3. **Game** - Game loop and persistence, orchestrates models + display
4. **Utils** - Reusable components
5. **Tests** - Mirror source structure, focus on core logic

## Save File Format

JSON file at `~/.tamagochi/save.json`:
```json
{
  "name": "Fluffy",
  "pet_type": "cat",
  "stats": {
    "hunger": 85,
    "happiness": 90,
    "energy": 75,
    "health": 100,
    "age": 1234,
    "level": 1
  },
  "state": "happy",
  "position": 45,
  "direction": 1,
  "created_at": "2025-10-25T10:30:00",
  "last_update": "2025-10-25T10:35:00",
  "eating_started_at": null,
  "accumulated_damage": 0.0,
  "accumulated_hunger_decay": 0.5,
  "accumulated_happiness_decay": 0.3,
  "accumulated_energy_decay": 0.2
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zefirior) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
