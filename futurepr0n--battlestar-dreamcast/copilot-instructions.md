## battlestar-dreamcast

> This document provides context for the Gemini code assistant to understand the Battlestar-Dreamcast project.

# GEMINI Code Assistant Context

This document provides context for the Gemini code assistant to understand the Battlestar-Dreamcast project.

## Project Overview

This is a 2D space shooter game for the Sega Dreamcast, developed using the KallistiOS (KOS) SDK. The game is written in C++ and features a classic arcade-style gameplay with a player-controlled ship, enemy waves, and a scoring system.

The project is structured around a central `GameEngine` class that manages the main game loop, including initialization, updates, rendering, and cleanup. The rendering is done using the low-level KallistiOS PVR library, with functions for drawing sprites, backgrounds, and text.

Key components of the project include:

*   **`main.cpp`**: The entry point of the application, containing the main game loop.
*   **`game_engine.cpp` / `game_engine.hpp`**: The core of the game, managing game states and systems.
*   **`renderer.cpp` / `renderer.hpp`**: Handles all rendering using the KallistiOS PVR library.
*   **`input_system.cpp` / `input_system.hpp`**: Manages player input from the Dreamcast controller.
*   **`wave_manager.cpp` / `wave_manager.hpp`**: Controls the spawning of enemy waves.
*   **`bullet_manager.cpp` / `bullet_manager.hpp`**: Manages player and enemy bullets.
*   **`audio_system.cpp` / `audio_system.h`**: Handles sound and music playback.
*   **`loadobj.cpp` / `loadobj.hpp`**: Responsible for loading game objects and their textures.
*   **`romdisk/`**: Contains the game assets, such as images and sounds, which are compiled into a ROM disk for the game.

## Building and Running

The project uses a `Makefile` for building and running the game. The following commands are available:

*   **`make`**: Compiles the project and creates the `battlestar.elf` executable.
*   **`make run`**: Builds the project and runs the game on a connected Dreamcast using `dc-tool-ip`.
*   **`make clean`**: Removes all compiled object files and the executable.

To send the game to a Dreamcast via the IP Tool, use the following command:

```bash
dc-tool-ip -x battlestar.elf -t <DREAMCAST_IP_ADDRESS>
```

## Development Conventions

*   The code is written in C++ and follows a class-based structure.
*   Header files (`.hpp`, `.h`) are used for class and function declarations.
*   The code makes extensive use of the KallistiOS API for hardware interaction.
*   Debugging information is printed to the console using the `DEBUG_PRINT` macro.
*   Performance is monitored using the `PERF_TIMER` macro and the `FrameRateCounter` class.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/futurepr0n)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/futurepr0n)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
