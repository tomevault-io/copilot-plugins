## spokes-godot

> Project instructions and conventions for Claude Code.

# CLAUDE.md - Spokes (Godot)

Project instructions and conventions for Claude Code.

## Project Overview
**Spokes** is a 2D Roguelike Cycling Simulator in Godot 4.6. It interfaces with smart trainers (Bluetooth FTMS) and features realistic physics and procedural map generation.

## Build & Test Commands
- **Run Game**: Open `project.godot` and press F5.
- **Run All Tests (Headless)**: 
  `godot --headless --path . -s addons/gut/gut_cmdln.gd -gdir=res://tests -gexit`
- **Run Specific Test File**: 
  `godot --headless --path . -s addons/gut/gut_cmdln.gd -gtest=res://tests/features/cycling/test_cyclist_physics.gd -gexit`
- **Local Web Deployment**: `./deploy_local.sh` (Injects hash, exports Web, restarts Docker).

## Coding Standards (Strict Static Typing)
- **Mandatory Static Typing**: Every variable, function parameter, and return value MUST have an explicit static type (e.g., `var x: float = 0.0`, `func f(a: int) -> void`).
- **Domain-Driven Directory Structure**:
  - `src/core/`: Global singletons (`SignalBus`, `Units`, `SettingsManager`).
  - `src/features/cycling/`: Physics, drafting, and `Cyclist` entity/components.
  - `src/features/map/`: Procedural generation and course profiles.
  - `src/features/progression/`: `RunManager`, `ContentRegistry`, rewards, items, FIT export.
  - `src/ui/`: Components and screens.
- **Conventions**:
  - Use `class_name` for all logic scripts to enable type-hinting.
  - Logic MUST be decoupled from SceneTree where possible (inherit from `RefCounted` or `Object`).
  - Use `SignalBus` for decoupled communication between managers and UI.
  - Units: Base calculations use Metric (m, m/s, kg). Use `Units` singleton for conversions.
  - Testing: Every feature script should have a corresponding test in `tests/` prefixed with `test_`.

## Key Architectural Patterns
- **Entity-Component**: `Cyclist.gd` coordinates modular child nodes (`HardwareReceiver`, `Drafting`, `Surge`, `Visuals`).
- **Resource-Based Data**: Data models (e.g., `CyclistStats`, `CourseProfile`, `EliteChallenge`) inherit from `Resource`.
- **Global Signal Bus**: UI and state managers communicate via `SignalBus.gd`.
- **Hardware Abstraction**: `TrainerService.gd` handles Web Bluetooth vs. Mock Desktop data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexkibler)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexkibler)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
