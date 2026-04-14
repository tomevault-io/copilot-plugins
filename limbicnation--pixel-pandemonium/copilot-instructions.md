## pixel-pandemonium

> This project is a fast-paced, abstract lo-fi FPS developed using **Godot Engine 4.3+**. It emphasizes momentum-based movement, visceral hitscan combat, and experimental "Trash Vision" mechanics.

# Pixel Pandemonium (Trash Vision) - Project Context

This project is a fast-paced, abstract lo-fi FPS developed using **Godot Engine 4.3+**. It emphasizes momentum-based movement, visceral hitscan combat, and experimental "Trash Vision" mechanics.

## Project Overview

*   **Core Concept:** A "trash game" aesthetic arena shooter where high-speed movement and aggressive combat are rewarded.
*   **Engine:** Godot 4.3+ (using the Forward+ renderer).
*   **Main Technologies:** GDScript, CSG Primitives (for geometry), and specialized shaders for a neon-on-black aesthetic.
*   **Architecture:**
    *   **Autoloads:** `GameManager` handles global game state, scoring, and state transitions.
    *   **Components:** Reusable logic is encapsulated in components like `HealthComponent` and `CameraShake`.
    *   **Entities:** Decoupled entities (Player, Target) that communicate via signals.

## Key Directories

*   `/scripts/entities/`: Core game logic for the player and targetable entities.
*   `/scripts/components/`: Modular logic (Health, Shake) attached to entities.
*   `/scripts/autoload/`: Global singletons (`GameManager`).
*   `/scenes/`: Scene files, with `test_level.tscn` as the primary development arena.
*   `/docs/`: Design documents, including the "Trash Vision" GDD and Player Controller specs.

## Building and Running

*   **Prerequisites:** Godot Engine 4.3 or higher.
*   **Main Scene:** `res://scenes/test_level.tscn`.
*   **Controls:**
    *   **WASD:** Momentum-based movement.
    *   **Shift:** Sprint (increases FOV).
    *   **Space:** Variable jump height.
    *   **Q / Ctrl:** Dodge/Dash.
    *   **LMB:** Fire (Hitscan).
    *   **RMB:** Aim (Precision mode).
*   **Execution:** Open the project in Godot and press **F5** to run the main scene.

## Development Conventions

*   **Global Classes:** Use `class_name` at the top of scripts for easy cross-referencing without manual loading.
*   **Decoupling:** Prefer signals for outward communication (e.g., `health_depleted`) and methods for inward interaction (e.g., `take_damage`).
*   **Configuration:** Use `@export` variables for gameplay constants (speeds, damage, etc.) to allow rapid balancing in the Godot Inspector.
*   **Snappy Physics:** Movement should avoid "floatiness." Acceleration and friction values are intentionally high (80+ and 60+ respectively).
*   **Component Pattern:** When adding new shared logic (e.g., an inventory or status effects), implement it as a separate `Node` (Component) that can be added to any entity.

## Implementation Details

### Movement System
Implemented in `player_controller.gd` using `CharacterBody3D`. It uses a `lerpf` with `1.0 - exp(-acceleration * delta)` for frame-rate independent snapping. Includes "Fast Fall" (pressing down in air) and "Coyote Time" for platforming feel.

### Combat System
Uses `direct_space_state.intersect_ray()` for instant hit detection. Damage is applied by calling `take_damage()` on the target's `HealthComponent`. Includes visual recoil, weapon bob, and procedural muzzle flashes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Limbicnation)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Limbicnation)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
