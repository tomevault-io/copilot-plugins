## pursue-ai-test

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bevy 0.17.3 game implementing Rain World-inspired AI state machine for creature pursuing player. State machine: Wander → Pursue → Search → Attack.

## Commands

### Build & Run
- `cargo run` - Build and run (debug)
- `cargo run --release` - Build and run (optimized)
- `cargo build` - Build only
- `cargo build --release` - Build optimized

### Testing & Quality
- `cargo test` - Run all tests
- `cargo test <test_name>` - Run specific test
- `cargo clippy` - Lint code
- `cargo clippy --fix` - Auto-fix lint warnings
- `cargo fmt` - Format code
- `cargo check` - Fast compilation check

### WASM Build
- Target: `wasm32-unknown-unknown` with `wasm-server-runner`
- Configured in `.cargo/config.toml`

## Architecture

### Bevy ECS Organization
- **Plugins**: Modular system organization (PathfindingPlugin, PursueAIPlugin, CollisionPlugin)
- **Systems**: Named with `s_` prefix (e.g., `s_input`, `s_collision`, `s_pursue_ai_update`)
- **Resources**: Global state (Level, PathfindingGraph, InputDir, GizmosVisible)
- **Components**: Entity data (Physics, PlatformerAI, PursueAI, GoalPoint)

### AI State Machine
Located in `src/ai/pursue_ai/`:
- **mod.rs**: State enum, plugin, main update system (`s_pursue_ai_update`)
- **wander.rs**: Wander state impl (currently only active state)
- **movement.rs**: Movement utilities
- States: Wander, Pursue, Search, Attack (only Wander implemented)
- State transitions handled via `Option<PursueAIState>` returns

### Pathfinding
- **a_star.rs**: A* algorithm impl
- **pathfinding.rs**: Graph structure, Bevy plugin
- Uses graph of nodes with edges, A* for navigation

### Level System
- **level.rs**: Polygon-based level generation
- Data loaded from `assets/level.json` via `include_bytes!` at compile-time
- Level represented as Vec<Polygon>, each with Vec<Vec2> vertices

### Collision System
- **collisions.rs**: Collision detection/handling
- Plugin pattern, runs before render system
- System ordering: `s_collision` → `s_render`

### Core Systems Order
```
s_input → s_move_goal_point → s_pursue_ai_update
s_collision → s_render
```

## Key Patterns

### System Registration
```rust
app.add_systems(Update, system_a.after(system_b))
app.add_systems(Update, (system_a, system_b).chain())
```

### Plugin Structure
```rust
pub struct MyPlugin;
impl Plugin for MyPlugin {
    fn build(&self, app: &mut App) {
        app.add_systems(Update, s_my_system);
    }
}
```

### Component Queries
```rust
Query<(&mut Transform, &mut Physics, &PursueAI)>
Query<&Transform, With<GoalPoint>>
```

## Constants
- `GRAVITY_STRENGTH: f32 = 0.5` (main.rs)
- `PURSUE_AI_AGENT_RADIUS: f32 = 8.0` (pursue_ai/mod.rs)

## Dependencies
- **bevy 0.17.3**: Game engine, ECS framework
  - Uses `ButtonInput` (not `Input`) for input handling
  - `PresentMode::AutoVsync` for VSync
- **rand 0.9**: RNG for pathfinding, color generation
- **serde/serde_json 1.0**: Level JSON deserialization

## Important Notes
- All systems prefixed with `s_`
- Bevy 0.17 has breaking changes from 0.16, check migration guide if updating
- Level data compiled into binary via `include_bytes!`
- Module structure follows Rust conventions (mod.rs as module root)
- State machine incomplete: only Wander state functional

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cmoyates)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cmoyates)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
