## byte90-alxvlabs

> - `src/` holds the firmware entry point (`src/main.cpp`).

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds the firmware entry point (`src/main.cpp`).
- `src/ProvisioningManager.*` manages provisioning config checks and activation flow.
- `lib/` stores reusable components (networking, audio, display, system, storage, etc.).
- `lib/mcp/` owns MCP server, tools, and registry (`McpServer`, `McpToolRegistry`).
- `lib/services/ApiClient.*` handles protocol JSON messages and device info payloads.
- `data/` contains runtime assets such as language packs (`data/lang`) and media (`data/sounds`, `data/gifs`).
- `docs/` provides design notes and conventions (see `docs/CODING_STYLE_GUIDE.md`).
- `test/` is reserved for PlatformIO unit tests.

## Architecture Overview
- Entry point in `src/main.cpp` wires together services, UI, audio, and network layers.
- Core application flow lives under `lib/core/`.
- Cross-cutting subsystems live in `lib/` (audio pipeline, networking, storage, system state, display/UI).
- Assets are loaded from `data/` via LittleFS; keep file paths stable when changing assets.
- Deep dives: `docs/AUDIO_PROCESSING.md`, `docs/AUDIO_STREAMING.md`, `docs/BUTTON_INTERACTION.md`, `docs/MEMORY_AND_CORE_ARCHITECTURE.md`, `docs/MCP_TOOLS_GUIDE.md`, `docs/API_REFERENCE.md`, `docs/XIAOZHI_MCPTOOL_API.md`.

## Build, Test, and Development Commands
- `pio run -e seeed_xiao_esp32s3` builds the firmware for the configured ESP32-S3 target.
- `pio run -e seeed_xiao_esp32s3 -t upload` builds and flashes to a connected device.
- `pio device monitor -b 115200` opens the serial monitor at the configured baud rate.
- `pio test -e seeed_xiao_esp32s3` runs PlatformIO unit tests in `test/` (if present).
- Do not run `pio run` or `pio run -t upload` in automation; the maintainer will build and flash locally.

## Coding Style & Naming Conventions
Follow `docs/CODING_STYLE_GUIDE.md`.
- Indentation: 4 spaces, no tabs; K&R braces.
- Files/classes: `PascalCase` (e.g., `NetworkClient.h`).
- Methods: `camelCase`; locals: `snake_case`; members: `_snake_case`.
- Use `#pragma once` in headers; keep lines ~80-100 chars (max 120).

## Testing Guidelines
- Framework: PlatformIO Test Runner (see `test/README`).
- Place tests under `test/` with clear, feature-based folder names.
- Name tests descriptively (e.g., `test_audio_pipeline.cpp`) and keep them hardware-appropriate.

## Commit & Pull Request Guidelines
- Use Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:` (examples: `feat: add gif player`, `fix: status bar drawing`).
- Use present tense, imperative mood; keep the subject under ~72 characters.
- PRs should describe changes, link relevant issues, and include logs or screenshots when behavior/UI changes.
- Suggested PR checklist:
  - Summary, testing notes, and risk/rollback notes if needed.
  - References to updated docs or assets.
  - Screenshots/GIFs for UI changes.

## Configuration Notes
- Target board and build flags live in `platformio.ini`; keep new flags documented there.
- When adding assets, place them under `data/` and reference paths explicitly in code.

---
> Source: [alxv2016/Byte90-alxvlabs](https://github.com/alxv2016/Byte90-alxvlabs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
