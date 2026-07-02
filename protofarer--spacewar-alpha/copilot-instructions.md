## spacewar-alpha

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spacewar! clone written in Odin using the Raylib graphics library. The game implements a classic arcade space combat game with realistic orbital mechanics around a central star.

## Build System & Development Commands

### Hot Reload Development (Recommended)
- `./build_hot_reload.sh` - Build the hot-reloadable game library
- `./build_hot_reload.sh run` - Build and run the hot-reloadable version
- `./game_hot_reload.bin` - Run the hot-reload executable (after building)

### Quick Development Tasks
- `./task.sh a` - Build hot reload
- `./task.sh b` - Run game
- `./task.sh c` - Build and run (equivalent to `./build_hot_reload.sh run`)

### Other Build Targets
- `./build_debug.sh` - Create debuggable build without hot reload
- `./build_release.sh` - Create optimized release build
- `./build_web.sh` - Build for web/WASM (requires Emscripten SDK)

## Architecture

### Hot Reload System
The project uses a sophisticated hot-reload architecture for fast development iteration:
- Main executable (`main_hot_reload.odin`) loads game logic as a shared library
- Game code (`src/game.odin` and related files) compiles to `build/hot_reload/game.so`
- When game code changes, the library is reloaded while preserving game state
- Memory tracking allocator detects leaks and bad frees during development

### Entry Points
- `src/main_release/` - Standard executable entry point for release builds
- `src/main_hot_reload/` - Hot-reload capable executable for development
- `src/main_web/` - WebAssembly entry point for web builds
- `src/game.odin` - Core game logic and main game loop

### Core Systems
- **Game State**: Main game logic in `game.odin` with entity management
- **Resource Management**: `resource_manager.odin` handles asset loading
- **Audio**: `audio_manager.odin` manages sound effects and music
- **Utils**: Platform-specific utilities in `utils_*.odin` files
- **Timers**: `timer.odin` provides game timing functionality

### Build Output Structure
- `build/hot_reload/` - Development builds with hot reload support
- `build/debug/` - Debug builds without hot reload
- `build/release/` - Optimized release builds
- `build/web/` - Web/WASM builds

### Platform Support
- Native builds for Linux, macOS, and Windows
- Web builds using Emscripten
- Platform-specific code handled through conditional compilation
- GPU preference hints for laptops (NvOptimus/AMD PowerXpress)

## Game Architecture
The game implements classic Spacewar! mechanics with:
- Two ships orbiting a central gravitational star
- Realistic physics simulation with orbital mechanics
- Fuel and torpedo management systems
- Round-based gameplay with scoring

---
> Source: [protofarer/spacewar-alpha](https://github.com/protofarer/spacewar-alpha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
