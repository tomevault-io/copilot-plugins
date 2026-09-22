## zombies

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Zombies++ is a top-down multiplayer zombie shooter written in C++17 for Windows, built as a Visual Studio 2022 solution. It uses a client/server architecture over ENet (UDP), OpenGL (GLFW + GLAD) for rendering, and Dear ImGui for menus.

## Building

This is an MSBuild solution — there is no CMake despite the stray `.vs/CMakeWorkspaceSettings.json`. Build from Visual Studio 2022 (open `Zombies++.sln`) or the command line:

```powershell
msbuild Zombies++.sln /p:Configuration=Debug /p:Platform=x64
```

- **Use the x64 platform.** The prebuilt libraries in `lib/` (`enet64.lib`, `glfw3.lib`) are 64-bit only; Win32/x86 configurations exist in the solution but will not link.
- Build order is handled by the solution: **Core** is a static library (`StaticLibrary`) that compiles to `lib/Core.lib`; **Client** and **Server** are `Application` targets that link against it.
- There is no test suite, linter, or CI in this repo.

### Running

- **Server** is headless and reads the bind IP from `argv[3]` (not `argv[1]` — `argv[1]`/`argv[2]` are ignored). It listens on UDP port **6969**.
- **Client** opens a 1280x720 window. The game loop only advances once `Network::startGame` is set (triggered by a `STARTGAME` command from the server).

### Include/library paths

- All include and library paths in the `.vcxproj` files are **relative**, anchored at `$(SolutionDir)` (the directory containing `Zombies++.sln`). This covers both the VC++ Directories (`IncludePath`/`LibraryPath`) and the per-config `AdditionalIncludeDirectories`. The solution builds from any checkout location — keep new paths `$(SolutionDir)`-relative rather than absolute.
- Third-party headers live in `include/` (enet, glm, glad, GLFW, KHR, stb_image); ImGui is vendored under `Client/src/ImGui/`. Prebuilt libs (`Core.lib`, `enet64.lib`, `glfw3.lib`) live in `lib/`.

## Architecture

Three projects share code through **Core**:

- **Core/** — Platform-agnostic game logic and the network wire format. Defines `GameObject` and its subclasses `Player`, `Enemy`, `Projectile`, plus the `Game` singleton. Both Client and Server compile against this.
- **Server/** — Authoritative simulation. Runs a tight `while(true)` loop: applies client input, spawns enemies, ticks all `GameObject`s, and broadcasts snapshots.
- **Client/** — Rendering and input. Runs the GLFW window loop, sends local input to the server, and interpolates remote state for display.

### The Game singleton and game objects

`Game` (`Core/src/Game.h`) is a static singleton holding `inline static std::vector<GameObject*> gameObjects`, the shared world. Both client and server iterate this vector each frame and call `update()` on active objects, dispatching by `dynamic_cast` to `Enemy`/`Player`/`Projectile`. Every networked object carries a `networkID` (unique across clients) and `clientID` (owner). New objects arriving over the network are staged in `pendingObjects`/`pendingUpdates` (guarded by `pendingMutex`) because networking runs on a separate thread.

### Networking model

Each side has its own `Network` class (`Server/src/Network.*` and `Client/src/Network.*`) — they are **separate implementations**, not shared, even though both are named `Network`. The network loop runs on a `std::thread` (`networkHost` on the server, `networkClient` on the client) while the main thread runs the game/render loop. Shared state crosses the thread boundary through the queues in `Game`/`Network`.

Wire format is built by `PacketBuilder` (`Core/src/PacketBuilder.h`) and is **raw struct memcpy** — not endian-safe or portable across compilers:

```
[uint32 tickID][uint8 packetCount]( [uint8 MessageType][raw struct bytes] )*
```

- `MessageType` (`CONNECT`/`DISCONNECT`/`OBJECT`/`CHAT`) tags each entry and selects which struct follows.
- The payload structs are `ObjectPacket`, `ClientPacket`, `CommandPacket`, unified as `PacketVariant = std::variant<...>`. `PacketBuilder::build()` `std::visit`s the variant to serialize.
- `CommandPacket` carries server→client control via `CommandType` (`SPAWNENEMY`, `STARTGAME`).

### Client-side interpolation

The client does **not** render server state directly. It buffers incoming `Snapshot`s (a `tickID` plus a list of `ObjectPacket`s) in `snapshotBuffer`/`snapshots`, then renders `interpolationDelayInTicks` (6) ticks in the past, lerping object positions between the two bracketing snapshots (`getSnapshotsForInterpolation` + `computeAlpha`). Tick rate is 60Hz. This is why `GameObject` carries `previousPos`/`targetPos`/`t`.

### Client rendering stack

The client is organized as a set of static singletons, each `init()`-ed once in `Client/src/main.cpp`: `Window`, `Camera`, `Renderer`, `ResourceManager`, `UIManager`, and `TileWorld`. `Renderer` draws textured quads via a single shared VAO/VBO and an orthographic projection; shaders are loaded from `Client/src/vertexShader.txt` and `fragmentShader.txt`. Sprites and tile textures are loaded from `Client/resources/` (paths are relative to the working directory at runtime).

---
> Source: [Acruas33/Zombies](https://github.com/Acruas33/Zombies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
