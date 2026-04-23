## snapi-assetpipeline

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Configure (from project root)
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build everything
cmake --build build

# Build specific targets
cmake --build build --target SnAPI.AssetPipeline       # Main shared library
cmake --build build --target SnAPI.AssetPipeline.Tests  # Tests
cmake --build build --target snpak                      # CLI tool

# Run all tests
cd build && ctest

# Run tests directly (with Catch2 options)
./build/SnAPI.AssetPipeline.Tests                      # All tests
./build/SnAPI.AssetPipeline.Tests "[uuid]"             # Tests by tag
./build/SnAPI.AssetPipeline.Tests "test name"          # Single test by name
```

CMake options: `SNAPI_BUILD_CLI`, `SNAPI_BUILD_TESTS`, `SNAPI_BUILD_PLUGINS`, `SNAPI_BUILD_EXAMPLES` (all ON by default).

## Architecture

The system is a game asset processing pipeline with two distinct phases:

**Build-time (Pipeline):** Source assets are processed through Import (parse source format) and Cook (optimize for runtime) phases, then written into `.snpak` pack files with optional LZ4/Zstd compression and XXH3-128 integrity checksums.

**Runtime:** `.snpak` files are mounted by `AssetManager`, which provides synchronous and async loading with LRU caching, ref-counted `AssetHandle<T>`, and priority-based thread pool loading.

### Module Layout

- `include/` - Public API headers (all `#include`-able by consumers)
- `src/Core/` - UUID system, PayloadRegistry, PipelineContext
- `src/Hashing/` - XXHash wrapper
- `src/Pack/` - SnPAK binary format reader/writer, compression, memory-mapped I/O
- `src/Pipeline/` - AssetPipeline engine, plugin loader (dlopen/LoadLibrary), SQLite incremental cache
- `src/Runtime/` - AssetManager, AssetCache (LRU/size-quota), AsyncLoader (thread pool with priorities)
- `cli/` - `snpak` command-line tool
- `plugins/Texture/` - Reference plugin (FreeImage-based importer, RGBA8 cooker)
- `tests/` - Catch2 unit tests

### Plugin System

Plugins are shared libraries loaded at runtime. They export a C function `SnAPI_RegisterAssetPipelinePlugin` (via `SNAPI_DEFINE_PLUGIN` macro) and register `IAssetImporter`, `IAssetCooker`, and `IPayloadSerializer` implementations through `IPluginRegistrar`.

### Key Types

- `Uuid` - Used as both AssetId and TypeId; can be deterministic (hash-based) or random
- `TypedPayload` - Type-identified binary blob (TypeId + data)
- `PipelineBuildConfig` - Configuration for pipeline builds (source roots, plugins, output, compression)
- `std::expected<T, std::string>` - Error handling pattern used throughout (C++23)

### SnPAK Binary Format

Header (magic "SNPAK\0\0\0") → Chunk Data (optionally compressed) → String Table → Type Table → Index Block (entries + checksums). See `SnPAK File Format Specification Version 1.md` for full details.

## Code Style

Configured via `.clang-format` (LLVM-based, 150 col, 2-space indent, braces on next line).

Naming conventions:
- `m_` prefix for member variables
- `b` prefix for booleans (e.g., `bSuccess`)
- `I` prefix for interfaces (e.g., `IAssetImporter`)
- `E` prefix for enums (e.g., `ELoadPriority`)
- `T` prefix for template type names
- UPPER_SNAKE_CASE for constants
- Pointer/ref alignment: left (`Type* p`, `Type& r`)

## Dependencies

All fetched automatically via CMake FetchContent (`cmake/Dependencies.cmake`): stduuid, xxHash, LZ4, Zstd, cereal, SQLite3, FreeImage (for Texture plugin), Catch2, GLFW (for examples).

## C++ Standard

C++23 required (`std::expected`). Extensions disabled.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RichmarIII) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
