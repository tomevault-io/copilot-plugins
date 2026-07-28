## onitama

> **Onitama** is a web-based implementation of the board game Onitama, featuring AI agents and multiplayer gameplay. The game is built in Rust and compiles to both native binaries and WebAssembly for browser play.

# Onitama - Claude Code Navigation Guide

## Project Overview

**Onitama** is a web-based implementation of the board game Onitama, featuring AI agents and multiplayer gameplay. The game is built in Rust and compiles to both native binaries and WebAssembly for browser play.

**Live Site**: https://onitama.app/

## Repository Structure

```
onitama/
├── onitamalib/          # Core game library (Rust)
│   ├── src/
│   │   ├── models.rs    # Core data structures (Board, Player, Card, Move, GameState)
│   │   ├── board.rs     # Board logic and move validation
│   │   ├── cards.rs     # Card definitions (43 cards across 3 sets)
│   │   ├── agents/      # AI implementations
│   │   │   ├── agents.rs      # AiAgent enum wrapper
│   │   │   ├── minimax.rs     # Minimax algorithm
│   │   │   ├── alphabeta.rs   # Alpha-beta pruning
│   │   │   ├── montecarlo.rs  # Monte Carlo tree search
│   │   │   ├── greedy.rs      # Greedy heuristic
│   │   │   ├── heuristics.rs  # Board evaluation
│   │   │   └── move_gen.rs    # Legal move generation
│   │   ├── gamemodes/   # Web-only game modes (feature-gated)
│   │   └── tests/       # Comprehensive unit tests (70+ tests)
│   └── Cargo.toml
├── onitamaserver/       # Multiplayer server (Actix-web)
│   ├── src/
│   │   ├── main.rs      # Server entry point
│   │   ├── rooms.rs     # WebSocket room management
│   │   └── routes.rs    # HTTP endpoints
│   └── Cargo.toml
├── static/              # Frontend assets
├── Dockerfile           # Container build
└── README.md
```

## Core Concepts

### Game Rules (Simplified)
- 5×5 board with 5 pieces per player (1 king, 4 pawns)
- Players have 2 cards; 1 spare card alternates between players
- Each card defines unique movement patterns
- Win by: capturing opponent's king OR moving your king to opponent's temple

### Key Data Structures

**Point** (`models.rs:30-80`)
- `x, y: i8` coordinates (0-4)
- Operations: Add, Sub, Neg, in_bounds(), invert()

**Player** (`models.rs:9-27`)
- Enum: `Red` (bottom, moves toward y=0) or `Blue` (top, moves toward y=4)
- Blue sees board inverted (movement deltas are negated)

**Card** (`models.rs:89-137`, `cards.rs`)
- 43 total cards across 3 sets: Base (16), Sensei's Path (16), Promotional (11)
- Each has moves (Vec<Point>) and direction (Left/Right/Balanced)
- Index-based serialization (0-42)

**Board** (`models.rs:185-194`)
- Contains: piece positions, player hands, spare card, current turn
- Immutable - `try_move()` returns new GameState

**Move** (`models.rs:198-201`)
- `Move { card, src, dst }` - move a piece
- `Discard { card }` - when no legal moves exist

**GameState** (`models.rs:205-208`)
- `Playing { board }` or `Finished { board, winner }`

## Building & Testing

### Build Commands
```bash
# Library only
cargo build -p onitamalib

# Library + server
cargo build

# WebAssembly (requires wasm-pack)
cd onitamalib
wasm-pack build --target web --features web

# Docker
docker build -t onitama .
```

### Testing
```bash
# Run all tests
cargo test --lib

# Run specific test module
cargo test --lib test_board

# Run with benchmark features (requires indicatif)
cargo test --lib --features benchmark

# Check compilation
cargo check --all-features
```

### Linting
```bash
# Run clippy
cargo clippy --all-targets --all-features

# Auto-fix issues
cargo clippy --fix --lib --allow-dirty
```

## Feature Flags

**onitamalib** (`onitamalib/Cargo.toml:56-71`)
- `web` - Enables WASM bindings and gamemodes for browser
- `agent` - Enables agent binary
- `benchmark` - Enables benchmark tests (requires indicatif dependency)

## Common Tasks

### Adding a New Card
1. Add variant to `Card` enum (`models.rs:89-137`)
2. Implement moves in `card.moves()` (`cards.rs:4-236`)
3. Implement direction in `card.direction()` (`cards.rs:237-286`)
4. Add to index mapping (`cards.rs:287-389`)
5. Add to appropriate CardSet (`cards.rs:391-446`)

### Adding Tests
- Model tests → `onitamalib/src/tests/test_models.rs`
- Board tests → `onitamalib/src/tests/test_board.rs`
- Game logic → `onitamalib/src/tests/test_game_logic.rs`
- Move generation → `onitamalib/src/tests/test_move_gen.rs`
- Register in `onitamalib/src/tests/mod.rs`

### Modifying AI Agents
- Agent enum → `onitamalib/src/agents/agents.rs`
- Algorithms in separate files under `onitamalib/src/agents/`
- Heuristics → `onitamalib/src/agents/heuristics.rs`
- Move ordering → `onitamalib/src/agents/move_gen.rs`

## Code Style & Conventions

### Recent Modernizations (2025)
- ✅ Use implicit returns (no unnecessary `return`)
- ✅ Use `&[T]` instead of `&Vec<T>` in parameters
- ✅ Implement `Display` instead of `ToString`
- ✅ Use `#[derive(Default)]` where appropriate
- ✅ Use `.is_empty()` instead of `.len() > 0`

### Patterns
- Board is **immutable** - operations return new GameState
- Red moves toward y=0, Blue toward y=4
- Blue player's moves are inverted (negated Point deltas)
- Feature-gated WASM code uses `cfg_if::cfg_if!`

## Known Issues & Gotchas

### Temple Win Condition Bug
**Location**: `board.rs:130-134`
**Issue**: There may be an edge case where pawns can win by reaching the temple (should only be kings)
**Test**: `test_game_logic.rs:95-135` (currently ignored)

### Benchmark Tests
Benchmark tests require the `indicatif` dependency and `benchmark` feature flag. They're feature-gated to avoid compilation errors.

### WASM Bindings
Some older WASM bindings (v0.2.81) will be rejected by future Rust versions. Consider upgrading when possible.

## Quick Reference

### Find Definitions
- **Win conditions**: `board.rs:125-137`
- **Move validation**: `board.rs:8-138`
- **Card moves**: `cards.rs:4-236`
- **Legal move generation**: `agents/move_gen.rs:5-44`
- **AI entry point**: `agents/agents.rs:18-26`

### Important Constants
- Board size: 5×5 (hardcoded in `models.rs:69`)
- Blue temple: `(2, 0)`
- Red temple: `(2, 4)`
- Total cards: 43
- Pieces per player: 5 (1 king + 4 pawns)

## Testing Philosophy

The test suite (70+ tests) focuses on:
1. **Model correctness**: Point math, card moves, player operations
2. **Board mechanics**: Initialization, move validation, captures
3. **Game logic**: Win conditions, state transitions
4. **Move generation**: Legal moves, edge cases, random selection

Tests use explicit board setups rather than random generation for reproducibility.

## Server Architecture

**onitamaserver** is an Actix-web server providing:
- WebSocket rooms for multiplayer
- REST API for room creation/joining
- Server-side AI evaluation (optional)

**Key files**:
- `main.rs` - Server setup and configuration
- `rooms.rs` - WebSocket connection management
- `routes.rs` - HTTP endpoints

## Development Tips

1. **Compile checks are fast**: Use `cargo check` for quick feedback
2. **Tests are comprehensive**: Run `cargo test --lib` before committing
3. **Feature flags matter**: Remember `--features web` for WASM code
4. **Board is immutable**: Never mutate, always return new state
5. **Red vs Blue perspective**: Blue's moves are inverted!

## Getting Help

- **Game rules**: https://www.arcanewonders.com/onitama/
- **Rust book**: https://doc.rust-lang.org/book/
- **Actix web**: https://actix.rs/
- **WASM pack**: https://rustwasm.github.io/wasm-pack/

---

**Last Updated**: January 2025
**Rust Edition**: 2018
**Test Coverage**: 70+ unit tests

---
> Source: [jackadamson/onitama](https://github.com/jackadamson/onitama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
