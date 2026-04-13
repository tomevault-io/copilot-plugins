## cpp-rpg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

- IMPORTANT: Only write safe code

## Project Overview

This is a top-down pixel RPG game (Zelda-like) written in C++ using raylib. The game features a player character that can move around the screen with proper boundary collision.

## Build System

The project uses CMake as its build system.

### Build Commands
```bash
# Create build directory
mkdir build && cd build

# Configure
cmake ..

# Build
make

# Run the game
./ZeldaGame
```

### Dependencies
- raylib (install instructions below)

#### Installing raylib on Ubuntu/Debian:

```bash
# Install dependencies
sudo apt install libasound2-dev libx11-dev libxrandr-dev libxi-dev libgl1-mesa-dev libglu1-mesa-dev libxcursor-dev libxinerama-dev libwayland-dev libxkbcommon-dev

# Install from source
sudo apt install cmake
git clone https://github.com/raysan5/raylib.git raylib
cd raylib
mkdir build && cd build
cmake -DBUILD_SHARED_LIBS=ON ..
make
sudo make install
sudo ldconfig
```

## Project Structure

```
cpp_game/
├── CMakeLists.txt      # CMake build configuration
├── README.md           # Project documentation
├── src/               # Source files
│   ├── main.cpp       # Entry point
│   ├── Game.cpp       # Main game class
│   ├── Game.h         # Game class header
│   ├── Player.cpp     # Player entity
│   └── Player.h       # Player header
├── include/           # Shared headers
│   └── Constants.h    # Game constants
└── assets/           # Game assets (sprites, sounds, etc.)

## Architecture

### Core Components
1. **Game Class**: Manages the main game loop, raylib initialization, and coordinates all game systems
2. **Player Class**: Handles player movement, input, and rendering
3. **Constants**: Centralized configuration for window size, colors, speeds, etc.

### Game Loop
The game uses raylib's built-in frame timing:
- Target: 60 FPS (set with SetTargetFPS)
- GetFrameTime() provides delta time for frame-rate independent movement
- BeginDrawing/EndDrawing for rendering

### Input System
- Continuous input polling using IsKeyDown
- Support for both Arrow keys and WASD
- Diagonal movement is properly normalized

### Movement & Boundaries
- Player moves at 200 pixels/second
- Boundary detection keeps player within window bounds
- Uses simple AABB collision detection with raylib's Rectangle struct

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DustyPolk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DustyPolk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
