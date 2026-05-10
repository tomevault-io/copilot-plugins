## breaktime

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BreakTime is a macOS menu bar app (Swift/SwiftUI) that reminds users to take breaks to prevent RSI. It's a **nudge, not an enforcer** — it encourages breaks but never forcefully prevents working.

## Build & Run

This is a native macOS app using Swift Package Manager (macOS 14+):

```bash
# Build
swift build

# Release build
swift build -c release

# Run
swift run

# Create .app bundle
bash scripts/build-app.sh
```

## Architecture

### Core Timer System

- Each break tier maintains its own independent active time counter. All counters increment simultaneously while the user provides keyboard/mouse input.
- Idle detection uses `CGEventSource.secondsSinceLastEventType` — after the idle threshold (default 3 min), counters **unwind** (decrement) at real-time speed.
- There is no distinction between idle, sleep, lock, or lid-closed — all are uniformly handled as "no input events."
- On wake from sleep, counters are bulk-unwound by the full idle duration (not 1s-per-tick).
- Timers start fresh at zero on app launch; timer state is not persisted.

### Break Tier Hierarchy

- Tiers are ranked by `breakDuration` (not `activeInterval`).
- When a break triggers, all tiers with equal or shorter `breakDuration` are cascade-reset to zero.
- **Merge threshold** (default 5 min): if a shorter break would fire but a longer break is due within the merge window, the shorter break is skipped.

### Break Flow Phases

1. **Warning** (30s): colored border on all screens, opacity ramps 0%→100%
2. **Overlay**: fullscreen dimmed/blurred overlay on all connected displays
3. **Typing grace period** (~5s): keyboard input passes through, break timer paused until user stops typing
4. **Break active**: timer counts down, keyboard input blocked via `BreakOverlayWindow` (swallows key events), action buttons available (skip, postpone 1/5/10 min, lock after break — one-way toggle by design)

### Exception System

- **Automatic**: microphone-in-use (detected via CoreAudio `kAudioDevicePropertyDeviceIsRunningSomewhere`), screen sharing active
- **User-configured**: per-app rules with `focused` or `opened` trigger modes
- While exceptions are active, breaks are **queued** (not suppressed — counters keep incrementing). When exceptions end, the longest queued break fires with a 30s warning first.
- When an exception starts during an active warning/break, the break is cancelled (overlay torn down) and the break is queued.

### Key macOS APIs

| Purpose | API |
|---------|-----|
| Idle detection | `CGEventSource.secondsSinceLastEventType(.combinedSessionState, ...)` |
| Sleep/wake | `NSWorkspace.willSleepNotification` / `didWakeNotification` |
| Screen lock | `com.apple.screenIsLocked` / `screenIsUnlocked` distributed notifications |
| Fullscreen overlay | `BreakOverlayWindow` (NSWindow subclass) at `.screenSaver` level, one per `NSScreen` |
| Lock computer | `SACLockScreenImmediate` via dlsym (private API) |
| Microphone detection | CoreAudio `kAudioDevicePropertyDeviceIsRunningSomewhere` |
| Menu bar | `NSStatusItem` |
| Launch at login | `SMAppService` |

### State Persistence

- **Persisted** (UserDefaults or file): break tier config, launch-at-login pref, exception app list
- **Not persisted**: timer state (fresh on launch), pause state (cleared on reboot/relaunch)

### Important Behaviors

- **Config changes during break**: If config is modified while a break/warning is active, the break is cancelled to avoid referencing stale tiers.
- **Display changes**: Warning borders and break overlays rebuild automatically when displays are added/removed (`didChangeScreenParametersNotification`).
- **Lock after break**: This is a one-way toggle by design — once enabled during a break, it cannot be undone (prevents circumvention).

### Permissions Required

Accessibility, Input Monitoring, and possibly Screen Recording. The app should guide users through granting these on first launch.

---
> Source: [benjick/breaktime](https://github.com/benjick/breaktime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
