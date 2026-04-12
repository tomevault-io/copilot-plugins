## typewell

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Typewell is a terminal-based typing practice application written in Haskell, similar to Keybr and Monkeytype. It features multiple game modes, progress tracking, and adaptive difficulty.

## Build Commands

```bash
# Build the project
cabal build

# Build and run
cabal run typewell

# Clean build artifacts
cabal clean
```

## Architecture

The project is structured as a library (`typewell`) with an executable (`typewell`):

```
app/Main.hs          - Entry point, calls UI.runApp
src/Typewell/
├── Types.hs         - Core data types (AppState, GameState, GameMode, etc.)
├── Game.hs          - Game logic (newGame, handleChar, handleBackspace)
├── UI.hs            - Brick TUI framework integration and event handling
├── UI/
│   ├── Themes.hs    - Color schemes and visual attributes
│   └── Widgets.hs   - UI widget rendering components
├── Stats.hs         - WPM and accuracy calculations
├── Progress.hs      - JSON persistence (~/.config/typewell/)
├── Adaptive.hs      - Weak key detection for adaptive mode
└── WordLists.hs     - Word list and text generation
```

### Key Data Flow

1. **Startup**: `Main.hs` → `UI.runApp` loads saved history/key stats, starts timer thread (100ms ticks), initializes Brick app
2. **Event Loop**: Brick handles keyboard events and timer ticks, routes to appropriate screen handlers
3. **Game Logic**: `Game.hs` processes character input, `Stats.hs` computes metrics
4. **Persistence**: On session completion, `Progress.hs` saves stats to JSON files

### Core Types (Typewell.Types)

- `AppState`: Main state with current screen, game state, history, key stats
- `GameState`: Target text, typed characters, timing, current mode
- `GameMode`: TimedMode, WordCountMode, EndlessMode, AdaptiveMode
- `Screen`: MenuScreen, ModeSelectScreen, TypingScreen, ResultsScreen, HistoryScreen

### Key Dependencies

- **brick** / **vty**: Terminal UI framework
- **aeson**: JSON serialization for persistence
- **microlens**: Lightweight lenses for state manipulation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuanw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuanw)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
