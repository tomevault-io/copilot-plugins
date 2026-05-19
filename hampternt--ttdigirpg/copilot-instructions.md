## ttdigirpg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ttdigirpg is an experimental Rust terminal application exploring how digital computation can enhance TTRPGs (tabletop role-playing games). The project philosophy is inspired by how Hearthstone leveraged computation to create mechanics that are imprcactical or impossible in physical card games. The goal is discovering what digital tools unlock for TTRPG design, not building a specific game.

The implementation uses World of Darkness-inspired mechanics as a testbed: 3 core attributes (Physical, Social, Mental) and skills divided into Talents, Skills, and Knowledges, all on a 1-5 dot scale representing realistic human capability.

## Build and Development Commands

```bash
# Build the project
cargo build

# Run the application (runs demo with character creation and database init)
cargo run

# Run tests
cargo test

# Run a specific test
cargo test test_name

# Run tests in a specific module
cargo test character::tests
```

## Code Architecture

### Module Structure

The codebase uses a non-standard module organization with code in `src/lib/` subdirectory:

- `src/lib.rs` - Library entry point, uses `#[path = "lib/..."]` attributes to reference lib/ subdirectory
- `src/main.rs` - Binary entry point, simply calls `demo()`
- `src/demo.rs` - Demo function showcasing character creation and database initialization
- `src/lib/entities/` - Game entity definitions (Character, Database, Economy)
- `src/lib/systems/` - Placeholder for future game systems (dice rolling, combat, etc.)
- `src/database/` - SQLite database storage location (created at runtime)

### Key Architectural Patterns

**Entity System**: Entities are currently simple structs with methods. Characters use direct field access (all public) with u32 types allowing flexibility beyond the typical 1-5 range.

**Database Layer**: The `Database` struct wraps rusqlite Connection. On initialization:
1. Checks if database file exists
2. Creates new database with tables if not found
3. Opens existing database otherwise
Currently implements basic table creation pattern (demo table only).

**Character System**: Characters default all stats to 1 (minimum human competency). The `Character::new()` constructor only takes a name - stats are modified after creation via direct field access. The `display()` method renders a formatted terminal character sheet using Unicode box-drawing characters.

### Dependencies

- `rusqlite` (0.38.0) - SQLite database with bundled feature for static linking
- `serde` (1.0) - Serialization framework with derive macros
- `serde_json` (1.0) - JSON serialization
- `uuid` (1.19) - UUID generation with v4 and serde support for EconomicEntity IDs

### Development Notes

**Experimental Nature**: This is a research project. Code in `entities/economy.rs` is incomplete (commented impl block, name field is i32 not String). The systems module is a placeholder with documentation outlining future features.

**Database**: Database is initialized on every run via demo(). The database path is `src/database/game_data.db` and will be created if it doesn't exist.

**Testing Philosophy**: Tests focus on core functionality (character defaults, stat modification). Tests use simple assertions on struct fields.

---
> Source: [Hampternt/ttdigirpg](https://github.com/Hampternt/ttdigirpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
