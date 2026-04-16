## candylauncher

> This file provides guidance to gemini (google.com/Gemini) when working with code in this repository.

# GEMINI.md

This file provides guidance to gemini (google.com/Gemini) when working with code in this repository.

## Build Commands

> IMPORTANT: No need to read any files, just execute the following command

### Generate, CMake with Ninja + MSVC
```shell
run_cmake_generate.cmd
```

### Build

```shell
run_build.cmd
```

### Test Commands
```shell
run_debug_test.cmd
```

## Architecture Overview

CandyLauncher is a lightweight Windows application launcher written in C++17 with the following core architecture:

### Core Components
- **CandyLauncher.cpp**: Main entry point and main seach list window management
- **ListedRunnerPlugin**: Core plugin system for managing searchable items
- **ListViewManager**: Handles the main list display and selection
- **TrayMenuManager**: System tray functionality
- **GlobalState**: Data persistence and configuration management and global variables

### List Indexer
The application centered around `ListedRunnerPlugin` which:
- Read `runner.json` configuration file and scan the files in the specified directory, and read the UWP app list.
- Provides fuzzy search capabilities using RapidFuzz library
- Supports Pinyin matching for Chinese characters (PinyinHelper)

### Search and Matching
- Uses RapidFuzz (3rd party library) for fuzzy string matching
- Custom Pinyin support for Chinese input via `PinyinHelper`
- Indexing system via `IndexedManager` for fast lookups

### UI Components
- **GlassHelper**: Handles Aero glass effects and window transparency
- **SettingWindow**: Configuration interface
- **EditManager**: Search input field management with keyboard shortcuts
- **ScrollViewHelper**: List scrolling functionality in the configuration interface
- **CustomButtonHelper**: beautiful custom buttons and radiobutton features
- **SwichView**: a switch control that imitates winui
- **SkinHelper**: Manage skin functions in the main window

### Keyboard Shortcuts
- **Escape**: Hide the launcher window
- **Ctrl+1 to Ctrl+9**: Quickly launch items 1-9 from the current filtered list
- **Enter**: Launch the currently selected item
- **Up/Down arrows**: Navigate through the list

### Configuration
- `runner.json`: Main configuration file for defining searchable items and directories
- `settings.json`: Application preferences items and defualt value
- `user_settings.json`: User-specific configuration overrides
- `skin_tests.json`: Interface skin configuration file

### Key Dependencies
- Windows APIs (Win32, GDI+, Shell)
- RapidFuzz for fuzzy matching
- Everything SDK for file system integration
- JSON handling via nlohmann/json (json.hpp)

### Build System
- Primary: CMake with support for Visual Studio, MinGW, and Ninja generators
- Supports both x64 and x86 architectures
- Uses precompiled headers (pch.h)
- C++17 standard requirement

### Non-important documents
- unuseful folder
- 3rdparty folder
- bkcode.txt
- LICENSE

## note

- Save files using UTF-8 with BOM
- No need to resolve warning reminder when building

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CandyTek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
