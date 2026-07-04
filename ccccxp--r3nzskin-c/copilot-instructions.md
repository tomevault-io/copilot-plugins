## r3nzskin-c

> This document provides guidelines for AI coding agents working on the R3nzSkin codebase.

# AGENTS.md - R3nzSkin

This document provides guidelines for AI coding agents working on the R3nzSkin codebase.

## Project Overview

R3nzSkin is a League of Legends skin changer that hooks into the game's DirectX 11 rendering pipeline. The project consists of two components:
- **R3nzSkin** - A DLL that injects into the game process to modify skins
- **R3nzSkin_Injector** - An executable that injects the DLL into the game

## Build Commands

### Prerequisites
- Visual Studio 2019 or 2022 with C++ workload
- Windows 10 SDK
- Platform Toolset v143

### Build Configurations
The solution supports four configurations:
- `RiotGamesServers|x64` - For international servers (64-bit)
- `RiotGamesServers|Win32` - For international servers (32-bit)
- `ChinaServer|x64` - For Chinese servers (64-bit)
- `ChinaServer|Win32` - For Chinese servers (32-bit)

### Command Line Build
```batch
# Build the main DLL (RiotGamesServers, x64)
msbuild R3nzSkin/R3nzSkin.vcxproj /p:Platform=x64 /p:Configuration=RiotGamesServers

# Build the main DLL (ChinaServer, x64)
msbuild R3nzSkin/R3nzSkin.vcxproj /p:Platform=x64 /p:Configuration=ChinaServer

# Build the injector
msbuild R3nzSkin_Injector/R3nzSkin_Injector.vcxproj /p:Platform=x64 /p:Configuration=RiotGamesServers

# Build entire solution
msbuild R3nzSkin.sln /p:Platform=x64 /p:Configuration=RiotGamesServers
```

### Output Directory
Build outputs are placed in `Release/<Configuration>/` (e.g., `Release/RiotGamesServers/`)

## Testing

This project does not have automated tests. Manual testing requires:
1. Building the DLL and injector
2. Launching League of Legends
3. Running the injector to inject the DLL
4. Pressing F7 in-game runs `testFunc()` in `Hooks.cpp` for debugging

## Code Style Guidelines

### Language Standard
- C++20 or later (`/std:c++latest`)
- Conformance mode enabled (`/permissive-`)
- No precompiled headers

### File Naming
- Header files: `PascalCase.hpp`
- Source files: `PascalCase.cpp`
- SDK headers in `SDK/` subdirectory

### Include Order
1. Standard library headers (`<cstdint>`, `<string>`, etc.)
2. Windows/DirectX headers (`<Windows.h>`, `<d3d11.h>`)
3. Third-party libraries (`"json/json.hpp"`, `"imgui/imgui.h"`)
4. Project headers (`"CheatManager.hpp"`, `"SDK/AIBaseCommon.hpp"`)

### Header Guards
Use `#pragma once` exclusively - no traditional include guards.

### Naming Conventions
- **Classes**: `PascalCase` (e.g., `CheatManager`, `AIBaseCommon`)
- **Functions/Methods**: `camelCase` with descriptive names (e.g., `get_character_data_stack`, `change_skin`)
- **Member variables**: `camelCase` (e.g., `localPlayer`, `heroList`)
- **Constants/Enums**: `PascalCase` or `SCREAMING_SNAKE_CASE` for macros
- **Namespaces**: `lowercase` or `snake_case` (e.g., `offsets`, `d3d_vtable`)
- **Template parameters**: `PascalCase` (e.g., `Type`, `ReturnType`)

### Type Usage
- Use fixed-width types: `std::int32_t`, `std::uint64_t`, `std::uintptr_t`
- Use `std::size_t` for sizes and indices
- Use `auto` with brace initialization: `auto value{ expression };`
- Prefer `const auto&` for loop iterables

### Function Declarations
- Mark non-throwing functions with `noexcept`
- Use `[[nodiscard]]` for getters and pure functions
- Use `[[maybe_unused]]` for intentionally unused parameters
- Prefer trailing return types with `[[nodiscard]]`

```cpp
[[nodiscard]] bool isLaneMinion() const noexcept;
[[nodiscard]] inline returnType name() const noexcept;
```

### Brace Style
- Opening brace on same line (K&R style for control structures)
- Member initialization uses brace initialization

```cpp
if (condition) {
    // code
}

class Example {
public:
    bool flag{ true };
    std::int32_t value{ 0 };
};
```

### Pointer/Reference Style
- `*` and `&` attach to type: `AIBaseCommon* player`
- Use `this->` explicitly for member access in methods
- Use `::` prefix for global Windows API calls: `::GetModuleHandleW()`

### Memory Management
- Use `std::unique_ptr` for owned resources
- Use raw pointers for non-owning references to game objects
- Memory offsets found via pattern scanning at runtime

### SDK/Game Object Patterns
Use helper macros defined in `SDK/Pad.hpp`:

```cpp
CLASS_GETTER(returnType, name, offset)     // Value getter
CLASS_GETTER_P(returnType, name, offset)   // Pointer getter
PAD(size)                                  // Padding bytes
```

### Hash Comparisons
Use FNV hashing for string comparisons:
```cpp
// Compile-time hash
if (hash == FNV("SRU_Baron"))

// Runtime hash
fnv::hash_runtime(player->get_character_data_stack()->base_skin.model.str)
```

### Error Handling
- Minimal exception usage (exceptions disabled in RiotGamesServers config)
- Use early returns with null/error checks
- Use `__try`/`__except` for SEH in critical paths

### Comments
- Avoid excessive comments - code should be self-documenting
- Use `//` for inline comments
- Document VTable offsets with pattern signatures

### Preprocessor Defines
- `_RIOT` - Defined for RiotGamesServers builds
- `_CRT_SECURE_NO_WARNINGS` - Suppress CRT security warnings
- `__SSE2__` - SSE2 instruction set enabled

## Key Architecture

### CheatManager (Singleton)
Central manager accessed via global `cheatManager` instance:
- `cheatManager.hooks` - DirectX hook management
- `cheatManager.config` - User configuration
- `cheatManager.gui` - ImGui interface
- `cheatManager.memory` - Memory/offset management
- `cheatManager.database` - Skin database
- `cheatManager.logger` - Logging system

### Offset Management
- Offsets defined in `offsets.hpp` with inline variables
- Pattern signatures in `Memory.hpp` for runtime scanning
- Use `offset_signature` struct for pattern definitions

### DirectX Hooking
- VMT hooking via `vmt_smart_hook.hpp`
- Hooks `IDXGISwapChain::Present` and `ResizeBuffers`
- ImGui rendering in hooked Present call

## Common Tasks

### Adding a New Offset
1. Add inline variable to appropriate namespace in `offsets.hpp`
2. Add pattern signature to `Memory.hpp` in `sigs` vector
3. Use offset via `offsets::Namespace::OffsetName`

### Adding a New Skin Feature
1. Modify `SkinDatabase.hpp` for skin data
2. Update `GUI.cpp` for UI elements
3. Handle skin changes in `Hooks.cpp`

### Modifying Game Object Access
1. Define offset in `offsets.hpp`
2. Add getter macro in appropriate SDK header
3. Access via `object->getter_name()`

---
> Source: [ccccxp/R3nzSkin-c](https://github.com/ccccxp/R3nzSkin-c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
