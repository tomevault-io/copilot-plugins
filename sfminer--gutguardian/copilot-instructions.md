## gutguardian

> This document contains key learnings, conventions, and architectural notes for the Gut Guardian project to ensure consistent and effective development.

# Gemini Project Notes: Gut Guardian

This document contains key learnings, conventions, and architectural notes for the Gut Guardian project to ensure consistent and effective development.

## Project Details

- **Engine:** Godot 4.5
- **Language:** GDScript

## Key Learnings & Conventions

### 1. Pause/Resume Logic

The pause functionality in this project has specific requirements due to its scene structure.

- **Core Mechanic:** The game is paused globally using `get_tree().paused = true`.
- **UI Layer:** The main UI is on a `CanvasLayer`. To ensure UI elements can respond when the game is paused, their `process_mode` must be managed carefully.
- **The `process_mode` Fix:** The final, working solution for the unpause functionality involved setting the `UnpauseButton`'s `process_mode` to `3` (Pausable). This is a counter-intuitive but critical detail for this specific project's pause behavior.

### 2. Debugging Conventions

To maintain a clean output log, all diagnostic or tracking `print` statements must be conditional.

- **Usage:** Wrap all such statements in an `if debug:` block.
- **Inheritance:** The `debug` variable is declared in the base `Microbe.gd` script and inherited by all child microbe classes (`Pathogen`, `Virus`, etc.).
- **Mutability:** The `debug_scr` variable in `Microbe.gd` is a `var` to allow it to be overridden in child classes for more granular debugging.

### 3. Microbe Lifecycle & Spawning

- **Data-Driven:** Microbe properties are defined in `MicrobeData` resources (`.tres` files) located in the `data/microbes/` directory.
- **Death/Disposal:** The `die()` function in `Microbe.gd` implements a "Containment Center" pattern. Upon death, a microbe is immediately moved to the offscreen position `(-1000, -1000)` and its collision shapes are disabled via `set_deferred()` to prevent physics errors.
- **Infection System:** Viral infection is handled via signals. The base `Microbe.gd` class defines the `infection_attempted` and `infected` signals. The `Pathogen.gd` script connects to these signals to update its specific UI elements (like the `InfectionCountLabel`).
- **Pathogen Population Cap:** The pathogen population was being artificially capped at 30 because the `_handle_attacking_state` in `Pathogen.gd` did not include a check for reproduction. Adding this check allows pathogens to reproduce even after they have started attacking the gut lining.

## Development Log & Key Fixes

### 1. Pathogen Population Cap at 30

-   **Problem:** Pathogen reproduction ceased once the population reached 30.
-   **Cause:** The `SimulationSettings.pathogen_attack_threshold` (set to 30) triggered a global "attacking" state for all pathogens. The logic within this state either blocked or starved the reproduction process.
-   **Fix:** The threshold-based global attack behavior was removed. Pathogens now act as individual opportunists, feeding on the gut lining when conditions are met, rather than entering a collective attack mode. This prevents reproduction from being starved by a global state change.

### 2. Centralized Reproduction via MicrobeManager

-   **Problem:** The previous reproduction logic caused errors due to duplicate `add_child()` calls and redundant signal emissions.
-   **Fix:** The `Microbe._attempt_reproduction()` function was refactored to be the single source of truth. It now exclusively calls `MicrobeManager.spawn_microbe()` to handle instantiation and scene tree management.
-   **New Logic:**
	1.  Parent microbe splits its energy.
	2.  A spawn position is calculated near the parent.
	3.  `MicrobeManager.spawn_microbe()` is called once to create the child, add it to the scene, and update tracking arrays.
	4.  The child's energy is set.
    5.  `microbe_reproduced` is emitted once.
    6.  A reproduction cooldown is initiated to prevent bursty feedback loops.

### 3. Mouseover Label Signal Connections

-   **Problem:** `mouse_entered` and `mouse_exited` signals were being connected multiple times (e.g., in both the editor and in code), causing errors.
-   **Fix:** All signal connections in `_ready()` (for mouse events and area detectors) are now guarded with `is_connected()` checks to prevent duplicate connections.

### 4. Resilient ActionClaims Integration

-   **Problem:** The game would crash on startup if the `ActionClaims` autoload singleton was not present.
-   **Fix:**
    1.  `ActionClaims` was officially added as a project autoload.
    2.  All calls to `ActionClaims` are now guarded (e.g., using `get_node_or_null("/root/ActionClaims")`) to make its presence optional and prevent errors.
    3.  A call to release claims was added to the `die()` function to prevent stale ownership on microbe death.

### 5. Energy Economy & Infection Rules

-   **Infection & Reproduction:** Reproduction is now explicitly blocked if a microbe `is_infected`.
-   **Reproduction Cooldown:** A `repro_cooldown_timer` was added to space out births and create more stable population dynamics.
-   **Food Energy:** The `consume_food()` function now correctly applies the `pellet_uptake_mult` to the energy gained from food pellets.
-   **Pathogen Energy Source:** The "energy stealing" mechanic was removed. Pathogens were gaining energy from nearby beneficial microbes, leading to an unfair advantage and population explosion. They now gain energy only from consuming food particles or by feeding on the gut lining.

### 6. Centralized Death & Cleanup

-   **Problem:** Previously, cleanup logic was scattered.
-   **Fix:** The death process is now centralized. When a microbe's `die()` function is called, it releases claims, disables collisions, and emits the `microbe_died` signal. The `MicrobeManager` listens for this signal and handles the actual object removal (`erase` from arrays, `remove_child`, and `queue_free`). The object pool is no longer in the active code path for births.

### 7. Intelligent Food Placement

-   **Problem:** Food pellets were spawning inside the gut lining, making them unreachable.
-   **Fix:** The `FoodSpawner` now derives its spawn area from the `lumen_nav_region` (a `NavigationRegion2D`), ensuring food only appears in the accessible lumen. A fallback to the viewport rect remains if the nav region is not found.

### 8. Population Chart UI

-   **Problem:** The population pie chart and its legend were overlapping and visually offset.
-   **Fix:** The UI was split into two separate controls: `PopulationPie` and `PopulationLegend`.
	-   Both subscribe to `MicrobeManager.population_changed` to stay in sync.
	-   The chart now correctly separates viruses from other pathogens.
	-   Drawing coordinates are now correctly calculated relative to the control's own rectangle, fixing the offset issue.
    -   `_get_minimum_size()` is implemented to ensure proper layout in the `VBoxContainer`.

### 9. Diagnostics & Tuning Levers

-   **Energy Snapshot:** A `_debug_energy_snapshot()` function was added to `MicrobeManager` to print the mean energy of pathogens, aiding in balancing.
-   **Tunable Parameters:** The following remain key levers for game balance:
    -   Reproduction threshold and cooldown.
    -   Food spawn rate and energy value.
    -   Virus infection chance and resistance mechanics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SFMiner)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SFMiner)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
