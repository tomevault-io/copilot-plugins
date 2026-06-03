## openwheel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Configure and build everything (from repo root)
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Build options (all default ON)
cmake -B build -DBUILD_DAEMON=ON -DBUILD_GADGET=ON -DBUILD_TESTS=ON

# Run tests
cd build && ctest --output-on-failure

# Binaries output to build/bin/
```

**Dependencies:** Qt6 (falls back to Qt5), KDE Frameworks 6 (optional), X11 XTest (optional), DBus. The gadget compiles with `NO_KDE_FRAMEWORKS` and without `HAVE_X11` in degraded mode.

## Architecture

OpenWheel is a two-process system for the ASUS Dial hardware input device on Linux.

### Signal Flow

```
ASUS Dial HID device (/dev/hidraw*)
    → openwheel-daemon (C, reads HID packets)
    → D-Bus signals on org.asus.dial (Rotate + Press)
    → openwheel-gadget (C++20/Qt6, processes and executes actions)
```

### Daemon (`openwheel-daemon/`)

C program that reads raw HID packets from the ASUS Dial and broadcasts rotation deltas (+1/-1) and button state (press/release) as D-Bus signals. Runs as a system daemon with syslog.

### Gadget (`openwheel-gadget/`)

Qt6/QML application with this internal flow:

- **DBusInterface** listens to daemon signals → emits Qt signals
- **DialController** orchestrates everything, exposed to QML via Q_PROPERTY/Q_INVOKABLE
- **RotationHandler** accumulates rotation deltas, applies velocity-based acceleration (exponential smoothing, max 4x), detects adjustment start/stop
- **ActionExecutor** executes actions: keyboard simulation (X11 XTest), mouse scroll, D-Bus method calls, shell commands
- **ProfileManager** loads JSON profiles from XDG data dirs, caches in QHash with shared_ptr
- **ApplicationMatcher** monitors active window via KWindowSystem, triggers automatic profile switching by matching window class/title regex or process name

### Profile System (`profiles/`)

JSON files defining per-application dial functions. Each function has clockwise/counterclockwise/click actions. Profiles match windows via regex patterns on window class, title, or process name. `system-default.json` is the fallback.

## Conditional Compilation

The gadget uses preprocessor guards extensively:
- `NO_KDE_FRAMEWORKS` — builds without KDE, loses window monitoring (default profile only)
- `HAVE_X11` — enables XTest keyboard/mouse simulation
- `QT_VERSION_MAJOR` — Qt5 vs Qt6 API differences (especially in KWindowSystem)

## CI

GitHub Actions runs cmake configure → build → ctest on `ubuntu-latest` for pushes/PRs to main.

---
> Source: [fredaime/openwheel](https://github.com/fredaime/openwheel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
