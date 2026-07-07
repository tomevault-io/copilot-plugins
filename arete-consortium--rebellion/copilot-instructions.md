## rebellion

> Space arcade shooter suite built with Rust and Bevy

# CLAUDE.md — rebellion

## Project Overview

Space arcade shooter suite built with Rust and Bevy

## Current State

- **Version**: 2.0.0
- **Language**: Rust
- **Files**: 333 across 6 languages
- **Lines**: 57,056

## Architecture

```
rebellion/
├── .cargo/
├── .github/
│   └── workflows/
├── assets/
│   ├── audio/
│   ├── backgrounds/
│   ├── fonts/
│   ├── models/
│   ├── powerups/
│   ├── ships/
│   └── sprites/
├── benches/
├── config/
├── docs/
│   └── harvested_from_python/
├── games/
│   └── caldari_gallente/
├── platforms/
│   ├── linux/
│   ├── macos/
│   ├── web/
│   └── windows/
├── src/
│   ├── assets/
│   ├── audio/
│   ├── campaigns/
│   ├── core/
│   ├── entities/
│   ├── esi/
│   ├── games/
│   ├── systems/
│   └── ui/
├── web/
│   └── assets/
├── .gitignore
├── CHANGELOG.md
├── CLAUDE.md
├── CONTRIBUTING.md
├── Cargo.lock
├── Cargo.toml
├── LICENSE
├── README.md
├── build-wasm.sh
├── colorize_sprites.sh
```

## Tech Stack

- **Language**: Rust, Python, TypeScript, Shell, JavaScript, HTML
- **Framework**: bevy, rust
- **Package Manager**: cargo
- **Linters**: clippy
- **Test Frameworks**: cargo test
- **CI/CD**: GitHub Actions

## Coding Standards

- **Naming**: snake_case
- **Line Length (p95)**: 72 characters

## Common Commands

```bash
# Build native release
cargo build --release

# Build WASM
bash build-wasm.sh

# Run native
cargo run --release

# Run tests
cargo test

# Lint
cargo clippy -- -D warnings

# Format
cargo fmt --check

# Benchmarks
cargo bench --bench game_systems

# Serve WASM locally (after build)
cd web && python3 -m http.server 8080
```

## Anti-Patterns (Do NOT Do)

- Do NOT commit secrets, API keys, or credentials
- Do NOT skip writing tests for new code
- Do NOT use `.unwrap()` in production code — use proper error handling
- Do NOT use `unsafe` without a safety comment
- Do NOT clone when a reference will do

## Dependencies

### Core
- bevy_egui
- serde
- serde_json
- rand
- fastrand
- image

## Domain Context

### Key Models/Classes
- `Ability`
- `AbilityActivatedEvent`
- `AbilityAura`
- `AbilityEffectParticle`
- `AbilityEffectType`
- `AbilityEffects`
- `AbilityEndedEvent`
- `AbilityIndicatorContainer`
- `AbilityIndicatorFill`
- `AbilityIndicatorText`
- `AbilityKeyHint`
- `AbilityPlugin`
- `AbilityType`
- `AbyssalDepthsPlugin`
- `AbyssalEnemyText`

### Domain Terms
- Abyssal Deadspace
- Activate Salt Miner
- Activate Ship Ability
- Active Buff Visuals
- Arrow Keys
- Authentic Visuals
- Barrel Roll
- Based Damage
- Building Requires Rust
- CCP

### Enums/Constants
- `AbilityEffectType`
- `AbilityType`
- `AbyssalRoom`
- `Achievement`
- `Act`
- `AmmoType`
- `BackButtonAction`
- `BackgroundShipClass`
- `BackgroundShipFaction`
- `BossProjectileStyle`

### Outstanding Items
- **TODO**: we could test for more things here, like `Set`s and `Map`s. (`web/eve_rebellion.js`)
- **TODO**: we could test for more things here, like `Set`s and `Map`s. (`web/rebellion.js`)

## AI Skills

**Installed**: 122 skills in `~/.claude/skills/`
- `a11y`, `accessibility-checker`, `agent-teams-orchestrator`, `align-debug`, `api-client`, `api-docs`, `api-tester`, `apple-dev-best-practices`, `arch`, `backup`, `brand-voice-architect`, `build`, `changelog`, `ci`, `cicd-pipeline`
- ... and 107 more

**Recommended bundles**: `full-stack-dev`

**Recommended skills** (not yet installed):
- `full-stack-dev`

## Git Conventions

- Commit messages: Conventional commits (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`)
- Branch naming: `feat/description`, `fix/description`
- Run tests before committing

---
> Source: [Arete-Consortium/rebellion](https://github.com/Arete-Consortium/rebellion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
