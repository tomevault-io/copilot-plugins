## survival-game

> A top-down 2D survival/crafting game built in Godot 4.3, inspired by Necesse. This is a parent-kid vibe coding project where a 7-year-old is making game design decisions.

# CLAUDE.md - Island Crafting Game

## Project Overview
A top-down 2D survival/crafting game built in Godot 4.3, inspired by Necesse. This is a parent-kid vibe coding project where a 7-year-old is making game design decisions.

## Key Context
- **Engine:** Godot 4.3+ (use GDScript, not C#)
- **Style:** Top-down 2D, pixel art aesthetic
- **Audience:** Code should be simple and well-commented for learning
- **Reference game:** Necesse (top-down survival/crafting with settlement building)

## Project Structure
```
island-game/
├── CLAUDE.md            ← You are here
├── PLAN.md              ← Master roadmap (13 phases)
├── PROGRESS.md          ← Track completed work
├── project.godot
├── scenes/
│   ├── main.tscn
│   ├── player/
│   ├── world/
│   ├── items/
│   ├── ui/
│   └── crafting/
├── scripts/
│   ├── autoload/        ← GameManager, Inventory, RecipeDatabase
│   ├── player/
│   ├── world/
│   ├── items/
│   └── crafting/
├── assets/
│   ├── sprites/
│   ├── sounds/
│   └── fonts/
└── resources/           ← Custom Resource files (.tres)
    ├── items/
    └── recipes/
```

## Workflow Rules

### Always Do First
1. Read `PLAN.md` to understand the full project vision
2. Read `PROGRESS.md` to see what's been completed
3. Identify which phase you're working on

### Always Do Last
1. Update `PROGRESS.md` with completed work
2. Output the next phase prompt for the user

### Use Sub-Agents
Spawn Opus 4.5 sub-agents for:
- Researching Godot 4.3 best practices before implementing
- Debugging complex issues
- Reviewing code for kid-friendliness
- Generating placeholder assets or test data

## Code Style Guidelines

### GDScript Conventions
```gdscript
# Use clear, descriptive names (a kid is reading this!)
var player_health: int = 100
var inventory_slots: Array = []

# Comment the "why", not just the "what"
# We use acceleration so movement feels smooth, not jerky
velocity = velocity.move_toward(direction * max_speed, acceleration * delta)

# Keep functions short and focused
func take_damage(amount: int) -> void:
    player_health -= amount
    if player_health <= 0:
        die()
```

### Scene Organization
- One scene per game object type
- Use descriptive node names: `Player`, `HealthBar`, `HitSound`
- Keep node trees shallow (max 3-4 levels)

### Signals Over Direct Calls
```gdscript
# Prefer this (loose coupling)
signal item_collected(item_name, count)

# Over this (tight coupling)
get_node("/root/Main/UI/Inventory").add_item(item)
```

## Godot 4.3 Specific Notes

### CharacterBody2D Movement
```gdscript
# Use move_and_slide() - no arguments in Godot 4
velocity = direction * speed
move_and_slide()
```

### Input Handling
```gdscript
# Define actions in Project Settings > Input Map
if Input.is_action_pressed("move_right"):
    direction.x += 1
```

### Resource Classes
```gdscript
# For items, tools, recipes - use custom Resources
class_name Item extends Resource

@export var item_name: String
@export var icon: Texture2D
@export var max_stack: int = 99
```

## Common Gotchas

| Problem | Solution |
|---------|----------|
| Blurry pixels | Project Settings → Rendering → Textures → Default Texture Filter → **Nearest** |
| Input not working | Check Input Map in Project Settings |
| Collision not detecting | Verify collision layers/masks match |
| Scene changes crash | Use `call_deferred("change_scene_to_file", path)` |
| UI behind game | Put UI in a `CanvasLayer` |

## Phase Checklist Reference

Quick reference for what each phase delivers:

- **Phase 1:** Project setup, GameManager autoload
- **Phase 2:** Player movement (8-dir WASD)
- **Phase 3:** Island tilemap with water collision
- **Phase 4:** Harvestable resources (trees, rocks, bushes)
- **Phase 5:** Inventory data system
- **Phase 6:** Inventory UI
- **Phase 7:** Tools and hotbar
- **Phase 8:** Crafting recipe data
- **Phase 9:** Crafting UI
- **Phase 10:** Placeable objects
- **Phase 11:** Day/night cycle
- **Phase 12:** Polish (sounds, particles)
- **Phase 13:** Title screen, menus, done!

## Testing Each Phase

After each phase, the game should still run. Test:
1. `F5` to run the main scene
2. Walk around (after Phase 2)
3. Interact with new features
4. Check for errors in Output panel

## Asset Placeholders

Until real art is added, use:
- **ColorRect** nodes for solid shapes
- **Polygon2D** for simple sprites
- Built-in Godot icons from `res://icon.svg`

Free assets when ready:
- Kenney.nl (search "Tiny Town" or "1-Bit")
- OpenGameArt.org
- itch.io free game assets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michaeljamie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
