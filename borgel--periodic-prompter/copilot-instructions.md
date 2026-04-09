## periodic-prompter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building the Application
```bash
# Build the macOS .app bundle
poetry run python build_app.py
# OR manually
poetry run python setup_simple.py py2app

# Launch the built app
open "dist/Periodic Prompter.app"
```

### Development
```bash
# Install dependencies
poetry install

# Run in development mode
poetry run python -m periodic_prompter.main

# Run with rumps (preferred)
poetry run python -m periodic_prompter.main_rumps
```

### Testing
Currently no formal test suite - testing is done manually by building and running the application.

## Architecture Overview

This is a macOS menu bar application that periodically prompts users about their work plans. The architecture follows a modular design with clear separation of concerns:

### Core Components

- **`main_rumps.py`** - Primary application entry point using rumps framework for native macOS menu bar integration
- **`scheduler.py`** - Background thread that triggers prompts at configured intervals using Python's `schedule` library
- **`notifications.py`** - Handles all user interactions via native macOS dialogs (AppleScript) with plyer fallback
- **`storage.py`** - JSON-based persistent storage for plans, history, and application state
- **`settings.py`** - Configuration management with validation and default values
- **`settings_gui.py`** - Tkinter-based GUI for configuration

### Key Architectural Patterns

1. **Component Coordination**: The main app coordinates between scheduler, notifications, and settings via dependency injection
2. **Callback-based Updates**: Scheduler notifies the menu bar app of plan updates through callbacks to prevent UI duplication
3. **Native macOS Integration**: Uses AppleScript for dialogs and rumps for menu bar to avoid cross-platform compatibility issues

### Build System

- **Poetry** for dependency management and Python packaging
- **py2app** for creating macOS .app bundles with proper framework exclusions
- **LSUIElement=True** configuration for background-only operation (no dock icon)
- **Custom build script** (`build_app.py`) that cleans and orchestrates the build process

### Data Storage

- Settings: `~/.config/periodic_prompter/settings.json`
- Plans: `~/.local/share/periodic_prompter/plans.json`
- Current state: `~/.local/share/periodic_prompter/current_state.json`
- Logs: User-configurable location

### Threading Model

- Main thread handles rumps menu bar UI
- Scheduler runs in daemon thread for background prompting
- Settings GUI runs in separate thread to avoid blocking main UI
- All user prompts run in separate threads to maintain responsiveness

## Important Implementation Details

### Menu Bar Updates
The menu bar title displays current plan text, but the menu structure is created only once during initialization. Use `update_menu_title()` to update the display, not `setup_menu()` which would duplicate menu items.

### Settings Window
Requires creating a hidden Tkinter root window due to rumps/Tkinter interaction. The settings GUI properly handles cleanup and mainloop management.

### Native macOS Integration
The application uses AppleScript for dialogs and native macOS notifications rather than cross-platform libraries to ensure proper macOS behavior and avoid deprecated framework issues.

### Build Configuration
The `setup_simple.py` file excludes problematic frameworks and includes all necessary dependencies. The build creates a complete .app bundle with proper macOS metadata and LSUIElement configuration.

## Configuration

The application is highly configurable through the Settings GUI:
- Prompt intervals (minimum 0.1 hours / 6 minutes)
- Working hours and weekday-only scheduling
- Logging and export options
- Plan persistence and history tracking

All settings are validated and have sensible defaults defined in `Settings.DEFAULT_SETTINGS`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/borgel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/borgel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
