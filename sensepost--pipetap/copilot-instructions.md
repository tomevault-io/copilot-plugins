## pipetap

> We are building a Windows named pipe debug proxy in C++, using a test client and server, a dll to inject that hooks named pipe related Windows API's using the Microsoft detours library, and a controlling GUI using ImGui in Visual Studio targeting only Windows. We are using vcpkg to manage most external dependencies, and cmake to compile for x86/x86. Once injected, the dll listens on two PID specific named pipes which are the control/events channels between the gui and the dll.

# Repository Guidelines

## Project Overview

We are building a Windows named pipe debug proxy in C++, using a test client and server, a dll to inject that hooks named pipe related Windows API's using the Microsoft detours library, and a controlling GUI using ImGui in Visual Studio targeting only Windows. We are using vcpkg to manage most external dependencies, and cmake to compile for x86/x86. Once injected, the dll listens on two PID specific named pipes which are the control/events channels between the gui and the dll.

### Intended Application Flow

- A real named pipe client talks to real named pipe server.
- A function intercepting DLL is injected into the real server, starts up the control named pipes and hooks the relevant named pipe related read/write API's. Without any GUI's connected, the real read/write API's are just called (even though they are hooked) and data flows freely.
- The GUI connects to the control channel named pipes, and messages sent using read/write windows API's are shown in the proxy feature. By default messages simply pass through. If editing is enabled in the GUI, requests and responses can be edited, and those edited values are sent back to the clients.

### General Agent Rules

When suggesting UI code, stick to the ImGui public api, and dont use the imgui_internal.h header directly. We are using ImGui 1.92.4. ImGui can be found here: <https://github.com/ocornut/imgui>.
Don't use any emojis in source code. Changes never have to be backwards compatible for any of the components. The final compiled artefact includes everything needed to function properly.

When proposing code, try and keep it as simple as possible. That is, whenever possible suggest changes that would be the most maintainable in the long term. Avoid small hacks as workarounds. Instead, suggest larger changes when needed. Keep the code DRY, avoiding duplicated code. If something should be refactored into a helper, do it.

## Project Structure & Module Organization

- `pipetap-gui/` hosts the Windows desktop client (Dear ImGui front-end) with UI logic in `app/` and entrypoint `main.cpp`.
- `pipetap-dll/` contains the injected support DLL.
- `pipetap-shared/` provides cross-target headers for logging, IPC contracts, and helpers reused by the GUI and DLL.
- `pipe-test-client/` and `pipe-test-server/` are minimal named-pipe fixtures for manual validation.
- `pipetap-python/` is the Python SDK published to PyPI; package sources live under `src/` and is used to interact with the TCP to named pipe proxy feature from "outside".
- Top-level `CMakeLists.txt` coordinates multi-target builds; presets live in `CMakePresets.json`.

## Coding Style & Naming Conventions

- C++: follow existing Visual Studio defaults—4-space indentation, brace-on-same-line, `PascalCase` for types, `snake_case` for functions like existing hooks. Respect the precompiled header includes at file tops.
- Python: mirror PEP 8 formatting, prefer `snake_case` for functions/modules, and keep public API surface documented in `pipetap-python/ README.md`.

---
> Source: [sensepost/pipetap](https://github.com/sensepost/pipetap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
