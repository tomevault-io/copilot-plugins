## corgibreak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CorgiBreak is a macOS menu bar app that implements the 20-20-20 eye care rule: every 20 minutes, look at something 20 feet away for 20 seconds. During breaks, a fullscreen blurred overlay appears with an animated pixel corgi and a countdown timer. Users can skip breaks with Esc or the Skip button.

## Build Commands

The project uses XcodeGen to generate the Xcode project from `project.yml`. The system `xcodebuild` may point to Command Line Tools instead of Xcode.app, so use the full path:

```bash
# Generate .xcodeproj from project.yml
xcodegen generate

# Build (uses Xcode.app's xcodebuild directly)
/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild \
  -project CorgiBreak.xcodeproj \
  -scheme CorgiBreak \
  -configuration Release \
  -derivedDataPath build \
  build

# Run
open build/Build/Products/Release/CorgiBreak.app

# Kill running instance
pkill -x CorgiBreak
```

After code changes: regenerate the project with `xcodegen generate` if files were added/removed, then rebuild. If only modifying existing files, rebuild is sufficient.

## Architecture

**App lifecycle**: SwiftUI `MenuBarExtra` (menu bar-only app, no dock icon via `LSUIElement=true` in Info.plist). `AppDelegate` disables sudden termination and prevents quit-on-last-window-close.

**Timer flow**: `TimerManager` (ObservableObject) drives everything — a 1-second `Timer` ticks down from 20:00. At zero, it creates an `OverlayManager` which spawns borderless `NSWindow`s at `.screenSaver` level on every screen. After 20 seconds (or skip), windows are removed with `orderOut` (not `close`, which would terminate the app).

**Overlay rendering**: `BreakView` (SwiftUI) is hosted inside each `NSWindow` via `NSHostingView`. It layers a `VisualEffectView` (NSVisualEffectView wrapper for blur) + semi-transparent black + the break UI content.

**Pixel corgi**: `PixelCorgi` loads pre-extracted PNG frames from the bundle (8 animations: jump, idle1, idle2, sit, walk, run, sniff, sniffwalk). Frames are extracted from `Resources/corgi-asset.png` sprite sheet using `scripts/extract_frames.py` (requires Pillow). A random animation is chosen each break, with continuous bounce.

**Key constraint**: Overlay windows must use `orderOut()` to hide, never `close()`. Using `close()` triggers macOS app termination since MenuBarExtra doesn't count as a window.

## Tech Stack

- Swift 5.9, SwiftUI, macOS 14.0+ deployment target
- XcodeGen for project generation (`project.yml`)
- No third-party dependencies

---
> Source: [VolodymyrM27/CorgiBreak](https://github.com/VolodymyrM27/CorgiBreak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
