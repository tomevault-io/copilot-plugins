## anemu

> <!-- ISU - Integrated System UI -->

<!-- ISU - Integrated System UI -->
<!-- AGENTS.md: Developer/LLM guide — architecture overview, module structure, key decisions, build commands -->

# ISU (Integrated System UI) - Agent Guide

## Project Overview
ISU is a unified emulation frontend for Android that integrates RetroArch and Eden/Yuzu. It presents a handheld console-style UI (Nintendo Switch / Steam Deck aesthetic) and acts as the only visible UI layer.

## Architecture
- **Language**: Kotlin with Jetpack Compose
- **Architecture**: MVVM with Clean Architecture layers
- **DI**: Hilt
- **Database**: Room
- **Navigation**: Jetpack Navigation Compose

## Module Structure
- `:app` - Main application, screens, viewmodels
- `:core:model` - Data models (Game, Platform, EmulatorType, etc.)
- `:core:database` - Room database, entities, DAOs
- `:core:emulator-bridge` - Emulator abstraction layer with RetroArch/Eden adapters
- `:core:ui` - Shared UI components and theme
- `:core:controller` - Controller detection and management

## Key Design Decisions
1. Emulator Bridge pattern: Each emulator implements `EmulatorBridge` interface
2. Game -> Emulator Mapping: Game entity stores platform + emulator + core info
3. Controller-first navigation: UI built for D-pad/joystick navigation
4. Room DB for game library persistence
5. DataStore for user preferences

## Build Commands
- `./gradlew assembleDebug` - Build debug APK
- `./gradlew assembleRelease` - Build release APK
- `./gradlew lint` - Run lint checks

## Adding New Emulators
1. Create adapter implementing `EmulatorBridge` in `:core:emulator-bridge`
2. Add `EmulatorType` enum entry
3. Register adapter in `EmulatorManager`
4. Add platform->emulator mappings in `RomScanner`

---
> Source: [gabe10926/anemu](https://github.com/gabe10926/anemu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
