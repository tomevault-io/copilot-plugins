## gomoku

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gomoku is a cross-platform desktop game built with Tauri v1.5 (React + TypeScript frontend, Rust backend). Players compete against AI opponents with an Elo rating system.

## Commands

This project uses **bun** as the package manager.

```bash
# Development
bun run dev              # Start Vite dev server (frontend only)
bun run tauri:dev        # Run full Tauri app in development mode

# Build
bun run build            # Build frontend (tsc + vite build)
bun run tauri:build      # Build production Tauri app

# Lint
bun run lint             # Run ESLint
```

## Architecture

### Frontend (src/)
- **App.tsx**: Single-file React application containing all UI components and state management
- Uses `@tauri-apps/api` to communicate with Rust backend via `invoke()` calls
- UI sections: Game board (Play), Save/Load, Rating system, AI Profiles, Self-Play calibration, User management

### Backend (src-tauri/src/)
- **main.rs**: Tauri command handlers and application state (`AppState` struct with Mutex-protected fields)
- **engine.rs**: `Board` and `GameState` structs for game logic
- **ai.rs**: AI move selection using negamax search with alpha-beta pruning; configurable via `AiConfig` (depth, max_candidates, randomness, max_nodes, defense_weight)
- **rules.rs**: `RuleSet` trait with `StandardRuleSet` implementation (5-in-a-row win condition)
- **rating.rs**: Elo rating system with 12 calibrated AI profiles, self-play calibration
- **users.rs**: Multi-user support with per-user ratings and settings
- **types.rs**: Shared type definitions (Player, Move, GameResult, etc.)

### Tauri Commands
Frontend communicates via these commands: `new_game`, `get_state`, `make_move`, `ai_move`, `save_game`, `load_game`, `export_training`, `get_ratings`, `get_users`, `set_active_profile`, `set_match_mode`, `create_user`, `set_active_user`, `delete_user`, `start_self_play`, `stop_self_play`

### Key Design Patterns
- State is shared via `AppState` with Mutex locks for thread safety
- Rating system uses two stores: `rating_base` (AI profile calibration) and `rating_user` (per-user adjustments)
- Self-play runs in background via `tauri::async_runtime::spawn_blocking` with progress events

---
> Source: [ZhangYiqun018/gomoku](https://github.com/ZhangYiqun018/gomoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
