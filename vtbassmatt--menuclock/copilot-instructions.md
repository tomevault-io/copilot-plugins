## menuclock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

### Building
```bash
# Debug build
swift build

# Release build
swift build -c release
```

### Running
```bash
# Run debug build
swift run

# Run release build
.build/release/MenuClock
```

### Testing
```bash
# Run all tests
swift test

# Run a specific test
swift test --filter MenuClockTests.testClockConfigWithAllFields
```

## Architecture Overview

MenuClock is a macOS menu bar application that displays world clocks. It's built as a Swift Package Manager (SPM) executable.

### Core Components

**main.swift** - Application entry point containing:
- `TwoLineStatusView`: Custom NSView that renders clock labels and times in a two-line format in the menu bar
- `MenuClockApp`: NSApplicationDelegate that manages the status item, configuration, menu, and timer updates

**Models.swift** - Configuration data models:
- `ClockConfig`: Represents a single clock with label, shortLabel, timeZone, format, and display mode (menubar/menu/both)
- `Config`: Top-level configuration with clocks array, updateInterval, and runAtStartup settings
- Both models have custom `init(from:)` decoders with fallback defaults and validation for malformed config files

### Key Design Patterns

1. **Display Filtering**: Clocks can appear in three places via the `display` property:
   - `menubar`: Only in the menu bar status item
   - `menu`: Only in the dropdown menu
   - `both`: In both locations (default)

   Helper methods `menuBarClocks()` and `menuClocks()` filter the clock list accordingly.

2. **Configuration Resilience**: The custom Codable implementations provide defaults for missing fields and print warnings for invalid values, allowing the app to handle slightly-malformed config files gracefully.

3. **Auto-Configuration**: If no config file exists at `~/Library/Application Support/MenuClock/config.yaml`, the app creates a default one with Seattle, Dublin, and Hyderabad time zones.

4. **Dynamic Updates**: The config can be reloaded at runtime via the menu, which rebuilds the menu items, recalculates the status item width, and restarts the timer with the new interval.

### Dependencies

- **Yams**: YAML parser/encoder for reading and writing configuration files

### Application Bundle Notes

The Package.swift includes linker flags to embed an Info.plist, which is necessary for the app to run as a proper macOS application (menu bar access, no dock icon via `.accessory` activation policy).

Login item registration via `SMAppService` only works when running from a .app bundle, not from the command line build. The code detects this and skips registration in development mode.

## Testing Strategy

Tests in `MenuClockTests.swift` cover:
- YAML parsing with all fields present
- Missing field handling and defaults
- Invalid field value handling
- Display mode filtering (menuBarClocks/menuClocks)
- Round-trip encoding/decoding
- Direct initialization of models

When adding features, write tests for the configuration model changes and any display filtering logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vtbassmatt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vtbassmatt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
