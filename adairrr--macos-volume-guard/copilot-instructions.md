## macos-volume-guard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VolumeGuard is a macOS menu bar application that prevents dangerous volume spikes when Bluetooth audio devices connect. It uses Apple's CoreAudio HAL APIs to monitor audio device changes and enforce per-device volume limits within ~15-30ms.

**Target:** macOS 13.0+ (Ventura), Swift 5.9+, SwiftUI

## Build Commands

```bash
# Generate Xcode project (if project.yml changed)
xcodegen generate

# Build debug
xcodebuild -scheme VolumeGuard -configuration Debug build

# Build release
xcodebuild -scheme VolumeGuard -configuration Release build

# Run the app (after building)
open ~/Library/Developer/Xcode/DerivedData/VolumeGuard-*/Build/Products/Debug/VolumeGuard.app
```

## Project Structure

```
VolumeGuard/
├── VolumeGuard/
│   ├── Core/
│   │   ├── CoreAudioInterface.swift   # HAL API wrapper
│   │   ├── AudioManager.swift         # Business logic, enforcement
│   │   └── SettingsManager.swift      # UserDefaults persistence
│   ├── Models/
│   │   ├── TransportType.swift        # Device type enum (bluetooth, usb, etc.)
│   │   ├── DeviceInfo.swift           # Runtime device state
│   │   ├── DeviceConfiguration.swift  # Stored config per device
│   │   └── AppSettings.swift          # Global settings
│   ├── Views/
│   │   ├── MenuBarView.swift          # Main popover UI
│   │   ├── DeviceRowView.swift        # Device list item with slider
│   │   └── SettingsView.swift         # Settings window (tabbed)
│   ├── VolumeGuardApp.swift           # App entry point, MenuBarExtra
│   ├── Info.plist                     # LSUIElement=true (no dock icon)
│   └── VolumeGuard.entitlements
├── project.yml                        # xcodegen spec
└── PLAN.md                            # Original PRD
```

## Architecture

### Core Components

- **CoreAudioInterface** - Thin Swift wrapper around CoreAudio C APIs (AudioHardware.h). Handles device enumeration, volume get/set, and property listeners via `AudioObjectAddPropertyListenerBlock`. No business logic.

- **AudioManager** - `@MainActor ObservableObject` that manages monitoring and enforcement. Uses value-based debouncing to prevent infinite loops when setting volume triggers callbacks.

- **SettingsManager** - Singleton handling UserDefaults persistence for `[String: DeviceConfiguration]` (keyed by device UID) and `AppSettings`.

### Data Flow

1. CoreAudio fires `kAudioHardwarePropertyDefaultOutputDevice` or `kAudioDevicePropertyVolumeScalar` notification
2. CoreAudioInterface callback invokes AudioManager handler on main thread
3. AudioManager compares current volume to limit, calls `setVolume()` if exceeded
4. Debouncing ignores the callback triggered by our own setVolume call

### Key Implementation Details

**Volume property fallback:** Some devices don't expose volume on `kAudioObjectPropertyElementMain`. Code tries element 0 first, then falls back to element 1.

**CFString handling:** Use `Unmanaged<CFString>` with `takeUnretainedValue()` for device name/UID properties to avoid memory warnings.

**Debouncing pattern:**
```swift
if let lastSet = lastSetVolume, abs(newVolume - lastSet) < 0.02 {
    lastSetVolume = nil
    return  // Ignore our own change
}
```

## Constraints

- Use only official Apple CoreAudio HAL APIs (AudioHardware.h)
- No third-party audio wrappers
- No polling - use property listeners exclusively
- App is LSUIElement (menu bar only, no Dock icon)
- Target macOS 13.0 minimum (use old `onChange(of:)` syntax without old/new params)

## Gotchas & Lessons Learned

**macOS 13 vs 14 API differences:**
- `onChange(of:) { oldValue, newValue in }` is macOS 14+ only. Use `onChange(of:) { newValue in }` for macOS 13.
- `@Environment(\.openSettings)` is macOS 14+ only. Don't use it.

**Opening windows from MenuBarExtra:**
- The `Settings` scene doesn't respond to `NSApp.sendAction(Selector(("showSettingsWindow:")))` reliably in menu bar apps.
- Solution: Use a `Window` scene with an ID instead, then call `openWindow(id: "settings")` via `@Environment(\.openWindow)`.
- Always call `NSApp.activate(ignoringOtherApps: true)` before opening windows from a menu bar app.

**CoreAudio Swift bridging:**
- Use `Unmanaged<CFString>` with `takeUnretainedValue()` for string properties (device name, UID) to avoid compiler warnings about forming UnsafeMutableRawPointer to Optional<CFString>.

**Build system:**
- Project uses xcodegen (`project.yml`) to generate the Xcode project
- Run `xcodegen generate` after modifying project.yml
- Build artifacts go to DerivedData, not the project folder

**SwiftUI Form layouts:**
- Forms apply their own styling and can interfere with nested HStack layouts, splitting grouped controls apart
- To keep controls grouped (e.g., icon + slider + label together on the right), use `LabeledContent`:
```swift
LabeledContent {
    HStack(spacing: 4) {
        Image(systemName: "lock.fill")
        Slider(value: $value, in: 0...1)
        Text("\(Int(value * 100))%")
    }
} label: {
    Toggle("", isOn: $enabled).labelsHidden()
}
```
- This properly separates the label (left) from the content group (right) within a Form

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adairrr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
