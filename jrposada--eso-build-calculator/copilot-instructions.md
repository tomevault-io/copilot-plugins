## eso-build-calculator

> A CLI tool for calculating and optimizing Elder Scrolls Online builds.

# ESO Build Calculator

A CLI tool for calculating and optimizing Elder Scrolls Online builds.

## Architecture

This project follows a Domain-Driven Design (DDD) approach adapted for a CLI application with static data.

### Layers

```
src/
├── cli/           # Entry points for commands
├── data/          # Static data (replaces database)
├── domain/        # Core domain entities and value objects
├── services/      # Orchestration layer
└── infrastructure/# Cross-cutting concerns
```

#### `cli/` - Command Entry Points

Entry points for CLI commands using `clap`. Each command (e.g., `optimize`, `view`) has its own module that parses arguments and delegates to services.

#### `data/` - Static Data Layer

Replaces a traditional database. Contains hardcoded game data.

Data is exposed as static references (`&'static`) via `Lazy` initialization.

#### `domain/` - Domain Layer

Core business entities and value objects. Contains no external dependencies.

#### `services/` - Orchestration Layer

Coordinates multiple domain entities and data sources.

#### `infrastructure/` - Cross-Cutting Concerns

Utilities shared across layers.

## Development

```bash
cargo build
cargo test
cargo run -- optimize --help
cargo run -- view --help
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jrposada)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jrposada)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
