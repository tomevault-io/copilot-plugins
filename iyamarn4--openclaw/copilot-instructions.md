## openclaw

> This file provides AI assistants with essential context about the OpenClaw codebase: its structure, build system, conventions, and development workflows.

# CLAUDE.md — OpenClaw

This file provides AI assistants with essential context about the OpenClaw codebase: its structure, build system, conventions, and development workflows.

---

## Project Overview

**OpenClaw** is a multiplatform C++ reimplementation of *Captain Claw* (1997), the classic Monolith platformer. The entire engine is written from scratch; it reuses only the original game assets from the `CLAW.REZ` archive (not redistributed). The project targets Windows, Linux, macOS, Android, and WebAssembly (Emscripten).

- **Language:** C++11
- **Renderer/Input/Audio:** SDL2, SDL2_image, SDL2_mixer, SDL2_ttf, SDL2_gfx
- **Physics:** Box2D
- **Data/Config:** TinyXML-2 (XML parsing)
- **Signals:** libsigc++3
- **Build system:** CMake (primary) + Visual Studio 2017 solution (`OpenClaw.sln`)
- **CI:** Travis CI (`.travis.yml`) and AppVeyor (`appveyor.yml`)

---

## Repository Layout

```
OpenClaw/                        # Root
├── Box2D/                       # Bundled Box2D physics library
├── Build_Release/               # CMake output directory (executables + assets)
├── ClawLauncher/                # GUI launcher (Windows/Linux via Mono)
├── libsigc++3/                  # Bundled signal/slot library
├── libwap/                      # Custom library for reading .REZ / .WAP assets
├── OpenClaw/                    # Main game source code
│   ├── Engine/                  # Core engine subsystems
│   │   ├── Actor/               # Actor + component system
│   │   │   └── Components/      # Individual actor components
│   │   ├── Audio/               # Audio manager
│   │   ├── Events/              # Event system
│   │   ├── GameApp/             # Application framework & main loop
│   │   ├── Graphics2D/          # 2D rendering
│   │   ├── Logger/              # Logging utilities
│   │   ├── Physics/             # Box2D integration
│   │   ├── Process/             # Process manager (game loop tasks)
│   │   ├── Resource/            # Resource cache & loaders
│   │   ├── Scene/               # Scene graph
│   │   ├── UserInterface/       # Menus, HUD, screens
│   │   └── Util/                # General utilities
│   ├── ActorController.cpp/h    # Player/actor input controller
│   ├── ClawEvents.cpp/h         # Game-specific event definitions
│   ├── ClawGameApp.cpp/h        # Top-level application class
│   ├── ClawGameLogic.cpp/h      # Game logic layer
│   ├── ClawHumanView.cpp/h      # Human (rendering) view
│   └── main.cpp                 # Entry point
├── Scripts/                     # Build/utility scripts
├── ThirdParty/                  # Additional third-party deps (e.g. TinyXML)
├── CMakeLists.txt               # Root CMake configuration
├── OpenClaw.sln                 # Visual Studio 2017 solution
├── .travis.yml                  # Travis CI config
└── appveyor.yml                 # AppVeyor CI config
```

---

## Architecture

### Application Layer

`ClawGameApp` inherits from `BaseGameApp` (in `Engine/GameApp/`) and overrides:
- `VGetGameTitle()` → `"Captain Claw"`
- `VGetGameAppDirectory()` → SDL2 `SDL_GetBasePath()`
- `VCreateGameAndView()` → instantiates `ClawGameLogic` + `ClawHumanView`

Entry point (`main.cpp`) calls `RunGameEngine(argc, argv)` which drives the main loop.

### Component-Entity System (Actor Model)

Actors are composed of components. Key files:

| File | Role |
|------|------|
| `Engine/Actor/Actor.h` | Base `Actor` class, holds components by type ID |
| `Engine/Actor/ActorComponent.h` | `ActorComponent` interface |
| `Engine/Actor/ActorFactory.h/.cpp` | Creates actors from XML definitions |
| `Engine/Actor/ActorTemplates.h/.cpp` | Predefined actor configs |
| `Engine/Actor/Components/` | Concrete component implementations |

Actor creation flow: XML element / resource path → `ActorFactory::CreateActor()` → component construction via `GenericObjectFactory` → unique `ActorId` (GUID counter).

### Interfaces

`Engine/Interfaces.h` defines the three main engine interfaces all major subsystems implement:
- `IGameLogic` — actor lifecycle, level/menu loading, game-state transitions
- `IGameView` — render tick, input routing, actor attachment
- `IGamePhysics` — add/remove bodies, apply forces, raycast/AABB queries

### Event System

Events are declared in `Engine/Events/` and game-specific ones in `ClawEvents.h`. The system uses `IEventData` base objects and a listener/dispatcher pattern.

### Physics

Box2D is bundled under `Box2D/` and integrated in `Engine/Physics/`. Collision categories are defined as `CollisionType`/`CollisionFlag`/`FixtureType` enums in `Interfaces.h`.

---

## Coding Conventions

### Naming
| Element | Convention | Example |
|---------|-----------|---------|
| Classes | `PascalCase` | `ClawGameLogic` |
| Virtual methods | `V` prefix + `PascalCase` | `VGetGameTitle()` |
| Member variables | `_camelCase` (underscore prefix) | `_lastActorGUID` |
| Constants / enums | `PascalCase` or `ALL_CAPS` | `ActorPrototype`, `SAFE_DELETE` |
| Files | `PascalCase` matching class name | `ActorFactory.h` |

### Pointers & Memory
- Raw owning pointers are wrapped with `SAFE_DELETE` / `SAFE_DELETE_ARRAY` macros (defined in `SharedDefines.h`).
- Shared ownership uses `std::shared_ptr`; aliases `StrongActorPtr`, `StrongActorComponentPtr` are used project-wide.
- Weak references use `std::weak_ptr`; `MakeStrongPtr()` helper safely promotes them.

### Types
`SharedDefines.h` provides fixed-width integer aliases — prefer these over raw `int`:
```cpp
uint64, uint32, uint16, uint8
int64,  int32,  int16,  int8
```

### XML / Data-Driven Design
Actor and level data are XML-based, parsed with TinyXML-2. Macros in `Engine/XmlMacros.h` standardise attribute access and error reporting.

### Platform Guards
Use `#ifdef _WIN32`, `#ifdef __ANDROID__`, `#ifdef __EMSCRIPTEN__` for platform-specific code. Android JNI bindings live in `main.cpp`.

---

## Building

### Prerequisites

**Linux (Ubuntu/Debian):**
```bash
sudo apt install cmake build-essential \
  libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev \
  libsdl2-ttf-dev libsdl2-gfx-dev \
  timidity freepats          # optional — background music
```

**Windows:** Visual Studio 2017 or NMake; SDL2/Box2D headers are bundled.

**macOS:** Homebrew SDL2 libraries + Xcode command-line tools.

**Android:** NDK + custom `Android.cmake` toolchain (see `CMakeLists.txt`).

**Emscripten (WebAssembly):** Emscripten SDK (`emcc`/`emcmake`).

### Standard (Linux / macOS / Windows NMake)
```bash
mkdir build && cd build
cmake ..                        # add -DCMAKE_BUILD_TYPE=Release for release
make -j$(nproc)                 # or: cmake --build .
```
Executables land in `Build_Release/`.

### Visual Studio 2017
```bash
mkdir build && cd build
cmake -G "Visual Studio 15 2017" ..
# Open OpenClaw.sln or: msbuild OpenClaw.sln
```

### Android
```bash
mkdir build && cd build
cmake -DAndroid=1 -DCMAKE_TOOLCHAIN_FILE=../Android.cmake ..
make
```

### WebAssembly (Emscripten)
```bash
mkdir build && cd build
emcmake cmake -DEmscripten=1 ..        # add -DExtern_Config=0 to bundle config.xml
make
# Deploy Build_Release/{openclaw.html,openclaw.js,openclaw.wasm,openclaw.data}
```

---

## Required Game Assets

The engine will not run without the original game assets. Place in `Build_Release/`:

| File | Description |
|------|-------------|
| `CLAW.REZ` | Original Captain Claw resource archive (~40 MB) |
| `ASSETS.ZIP` | Community/custom assets zip |

These files are **not** included in the repository (copyright Monolith).

---

## Development Workflow

1. **Branch from `master`** for new features; use short descriptive names.
2. **Build with CMake** in a separate `build/` directory (out-of-source builds only).
3. **Test on Linux** as the primary CI target (Travis CI runs `./travis.sh` → `cmake .. && make`).
4. **Add new actors/components via XML** where possible; avoid hardcoding actor parameters.
5. **Register new event types** in `ClawEvents.h` and wire up listeners in the appropriate view or logic class.
6. **New components** go in `Engine/Actor/Components/` and must be registered with `ActorFactory`.
7. Keep platform-specific code isolated behind preprocessor guards.

---

## CI / Testing

- **Travis CI** builds on Linux (GCC + Clang), macOS, Android, and Emscripten.
- **AppVeyor** handles Windows MSVC builds.
- There is no automated game-logic test suite; CI validates successful compilation only.
- Coverity Scan integration is configured for static analysis on select branches.

Build script: `./travis.sh` (wraps CMake configure + build steps for CI).

---

## Known Limitations & TODOs

- WebAssembly builds do not support `.xmi` (MIDI) audio — search `TODO: [EMSCRIPTEN]` for affected code.
- Death and teleport fade-in effects are broken under Emscripten (buffer cleared each frame).
- Some `SDL_Mixer` functions are not yet implemented in the Emscripten build.
- Not all 14 levels are fully playable — work in progress.

---

## Key Files Quick Reference

| File | Purpose |
|------|---------|
| `OpenClaw/main.cpp` | Entry point, platform bootstrapping |
| `OpenClaw/ClawGameApp.h/.cpp` | Top-level app class |
| `OpenClaw/ClawGameLogic.h/.cpp` | Core game logic |
| `OpenClaw/ClawHumanView.h/.cpp` | Rendering/input view |
| `OpenClaw/Engine/Interfaces.h` | All major engine interfaces |
| `OpenClaw/Engine/SharedDefines.h` | Type aliases, macros, SAFE_DELETE |
| `OpenClaw/Engine/Actor/ActorFactory.h` | Actor instantiation |
| `OpenClaw/Engine/Actor/ActorComponent.h` | Component interface |
| `CMakeLists.txt` | Build configuration |
| `Build_Release/config.xml` | Runtime configuration file |

---

*This file was generated to assist AI coding tools. Keep it updated as the project evolves.*

---
> Source: [iyamarn4/OpenClaw](https://github.com/iyamarn4/OpenClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
