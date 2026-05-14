## lullaby

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test

Build (Debug):
```bash
cmake -B out/build/x64-Debug
cmake --build out/build/x64-Debug
```

Run tests:
```bash
ctest                    # Run all tests
ctest -R MusicService    # Run specific test suite by pattern
```

## Running Locally

```bash
cd out/build/x64-Debug
./Lullabies.exe 8080 ../../..
```

Access the web UI:
- Player: http://localhost:8080
- Admin (song management): http://localhost:8080/admin.html

## Git Workflow

- For each feature or fix, first checkout and pull main, then create a new branch
- Commit regularly with incremental checkpoints to enable easy reverts
- Do not squash commits during development - the user will squash before pushing

## Architecture

Layered C++ application serving a web UI for music playback:

```
Browser <--HTTP/SSE--> MusicController --> MusicService --> MusicRepository --> SQLite
                         |                    |
                         v                    v
                   MusicSerializer      AudioPlayer --> miniaudio --> Bluetooth
```

**Layers:**
- `MusicController` - HTTP API endpoints (play, pause, list, SSE)
- `MusicService` - Core playback logic, song selection, state management
- `MusicRepository` - SQLite data access
- `AudioPlayer` - miniaudio wrapper for playback
- `MusicSerializer` - JSON serialization
- `MusicStatusUpdater` - SSE broadcast to clients

**Key design:** `lullaby_core` library is separate from `AudioPlayer` implementation, allowing tests to mock audio via `UNIT_TEST` preprocessor flag.

## Tech Stack

- C++23, CMake
- httplib (HTTP server), nlohmann/json, miniaudio, SQLite3
- GTest for unit tests
- Server-Sent Events for real-time UI updates

---
> Source: [m4wright/lullaby](https://github.com/m4wright/lullaby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
