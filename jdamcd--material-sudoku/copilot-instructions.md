## material-sudoku

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build
./gradlew app:assembleGoogleDebug

# Lint & formatting
./gradlew spotlessCheck app:lint

# Auto-fix formatting
./gradlew spotlessApply

# Tests
./gradlew core:test                # Core module unit tests
./gradlew app:testGoogleDebugUnitTest  # App module unit tests

# Run a single test class
./gradlew core:test --tests "com.jdamcd.sudoku.game.GameTest"
```

## Architecture

Multi-module Android app (Kotlin) — a Material Design sudoku game.

**Modules:**
- **core** — Pure Kotlin module with game logic, solver (Dancing Links algorithm), and utilities. No Android dependencies.
- **app** — Main Android application. Two product flavors: `google` (Play Services) and `vanilla` (no Play Services).
- **solver** — Standalone sudoku solver Android app, depends on core.

**Key patterns:**
- **Hilt** for dependency injection (`@HiltAndroidApp`, `@HiltViewModel`, `@AndroidEntryPoint`)
- **RxJava 2** for async data flows in the repository layer
- **Room** for SQLite persistence, with a pre-populated asset database
- **ViewModel + LiveData** for UI state (puzzle screen); MVP presenters elsewhere (browse, scoreboard)
- Custom `PuzzleView` (Canvas-based) hierarchy: `GamePuzzleView` (interactive) and `PreviewPuzzleView` (read-only)

**Data flow:** Fragment → ViewModel/Presenter → PuzzleRepository (RxJava) → PuzzleDao (Room) → SQLite

**Game state:** `Game` class in core manages answers grid, candidate notes, and undo/redo via a move history stack (`Answer`, `Note`, `Clear` move types). `Sudoku` holds the puzzle definition (givens + solution). Grids are serialized as compact strings for database storage via `Format` utilities.

## Code Style

Enforced by Spotless with ktlint 0.43.2. Run `./gradlew spotlessApply` to auto-format before committing.

---
> Source: [jdamcd/material-sudoku](https://github.com/jdamcd/material-sudoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
