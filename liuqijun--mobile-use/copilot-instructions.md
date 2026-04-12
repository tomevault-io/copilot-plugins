## mobile-use

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Release build

# Run
cargo run -- <args>            # Run with arguments
./target/debug/mobile-use <args> # Run built binary

# Test
cargo test                     # Run all tests
cargo test <test_name>         # Run specific test
cargo test -- --nocapture      # Show println output

# Check without building
cargo check

# Enable debug logging
RUST_LOG=debug cargo run -- <args>
```

## Architecture

### Daemon-Based CLI Architecture

```
CLI Command → Unix Socket → Daemon Process → WebSocket → Flutter VM Service
                              ↓
                         Session Manager (in-memory state)
```

The daemon (`~/.cache/mobile-use/daemon.sock`) maintains persistent WebSocket connections and session state. CLI commands are stateless - they communicate with the daemon via Unix socket IPC.

### Module Structure

- **`src/main.rs`** - Command dispatch and most action implementations. The majority of command logic lives here, not in the `commands/` module.
- **`src/daemon/`** - Background daemon process
  - `server.rs` - Unix socket server, request handling
  - `client.rs` - CLI-side daemon client
  - `session_manager.rs` - WebSocket connections and session state
  - `protocol.rs` - `DaemonRequest`/`DaemonResponse` types
- **`src/platform/flutter/`** - Flutter integration
  - `vm_service.rs` - Flutter VM Service WebSocket JSON-RPC 2.0 client
  - `semantics.rs` - Parse Flutter semantics tree into `ElementNode` tree with style extraction
  - `process_manager.rs` - Spawn and manage `flutter run --machine` process for `run` command
- **`src/platform/android/`** - Android integration
  - `adb.rs` - ADB command wrapper for device interaction (tap, input, screenshot)
  - `uiautomator.rs` - Parse Android uiautomator XML into element tree
- **`src/platform/ios/`** - iOS integration
  - `wda.rs` - WebDriverAgent HTTP client implementing `DeviceOperator` trait
  - `wda_manager.rs` - WDA lifecycle management (build, install, launch, port forwarding)
  - `elements.rs` - Parse WDA JSON source into element tree
  - `discovery.rs` - iOS device discovery via `idevice_id`
- **`src/core/types.rs`** - Core types: `ElementNode`, `ElementRef`, `Bounds`, `StyleInfo`, `RefMap`, `DeviceOperator` trait
- **`src/cli/`** - CLI argument parsing (`parser.rs`) and output formatting (`output.rs`)

### Key Types

```rust
// Element with bounds and style (stored in daemon, used for tap/input)
ElementRef { ref_id, element_type, label, bounds, properties, style }

// Hierarchical element tree (returned by elements command)
ElementNode { ref_id, element_type, label, bounds, properties, style, children }

// Visual style info for design comparison
StyleInfo { text_color, font_size, font_weight, background_color, border_radius, ... }
```

## Flutter Integration Details

### Run Command (Recommended)

The `mobile-use run` command wraps `flutter run --machine` and handles connection automatically:

```bash
cd your_flutter_app
mobile-use run -- -d <device_id>
```

This spawns `flutter run --machine`, parses JSON output to extract VM Service URL, auto-connects to daemon, and passes stdin for interactive commands (r/R/q).

### VM Service Connection (Manual)

Flutter debug apps expose a VM Service WebSocket. URL format conversion:
```
flutter run output:  http://127.0.0.1:55370/jcsm3VFShF0=/
mobile-use requires:   ws://127.0.0.1:55370/jcsm3VFShF0=/ws
                     ↑                                  ↑
                  http→ws                          append /ws
```

For remote devices (wireless ADB), port forwarding is required:
```bash
adb -s 192.168.1.100:5555 forward tcp:55370 tcp:55370
```

### Coordinate System (Critical)

In `semantics.rs`, the `parse_text_node_scaled` function handles coordinate transformation:

1. **Device Pixel Ratio**: Flutter returns logical pixels; multiply by scale factor (e.g., 2.8x) for physical pixels
2. **Relative Coordinates in Scrollable Containers**: Child elements inside scrollable containers (`hasImplicitScrolling` flag) report coordinates relative to the scroll viewport, NOT absolute screen coordinates. The parser accumulates `parent_offset_x/y` through recursion to compute absolute positions.

```rust
// Child absolute position = child relative position + accumulated parent offset
let (child_offset_x, child_offset_y) = (parent_offset_x + lb.x, parent_offset_y + lb.y);
```

This was a critical bug fix - without offset accumulation, `input` and `tap` commands hit wrong coordinates for elements inside ListView/ScrollView.

### Elements Command

```bash
mobile-use elements           # Get full element tree with styles
mobile-use elements -i        # Interactive elements only
```

### Flutter Commands

```bash
mobile-use flutter reload     # Hot reload the app
mobile-use flutter restart    # Hot restart the app
mobile-use flutter widgets    # Get widget tree
```

## iOS Integration Details

### DeviceOperator Trait

All platform-specific operations (tap, swipe, input, screenshot) are abstracted behind `DeviceOperator` trait in `core/types.rs`. Both `AdbClient` (Android) and `WdaClient` (iOS) implement this trait.

### iOS Setup Flow

```bash
mobile-use setup-ios --team-id TEAM_ID   # Build & install WDA (one-time per device)
mobile-use connect-ios --team-id TEAM_ID  # Launch WDA, start iproxy, connect
```

### WDA Coordinate System

iOS WDA works in logical points. `WdaClient` detects the device scale factor and converts physical pixel coordinates (used by mobile-use) to logical points for WDA actions via `to_points()`.

### iOS Element Properties

WDA returns properties as strings (`"0"`/`"1"` instead of booleans). The iOS element parser in `elements.rs` handles this conversion:
- `isEnabled` → `enabled` property (stored only when false)
- `isFocused` → `focused` property (stored only when true)
- Switch `value` "0"/"1" → `isToggled` property

## Common Pitfalls

1. **`text` command already taps to focus** - Don't add extra `tap` before `text`. The `input_action` function in `main.rs` already does: tap → wait 100ms → input text.

2. **VM Service URL changes on app restart** - Each `flutter run` generates a new token. Must reconnect with new URL.

3. **Wireless ADB limitations** - Some operations (scroll, swipe) may fail with `INJECT_EVENTS` permission error on wireless debugging. Use USB connection for full functionality.

4. **Element refs are ephemeral** - `@e5` from one `elements` call may refer to different element after UI changes. Always re-fetch before operating.

5. **Scroll direction is content-based** - `scroll down` means show content below (finger swipes up). The implementation inverts the swipe direction relative to the user-specified scroll direction.

6. **Platform property naming** - `is enabled/checked/focused` checks multiple property names across platforms (Flutter: `hasEnabledState`+`isEnabled`, Android: `isDisabled`, iOS: `enabled`). See the `is_state` match block in `main.rs`.

## Test Apps

- **`test_app/`** - Flutter app for testing. Covers buttons, text inputs, scrollable lists, form controls.
- **`test_app_ios/`** - Native SwiftUI app (same UI pages) for iOS-specific testing.
- **`test_app_android/`** - Native Android apps (Compose + View variants) for Android-specific testing.

```bash
# Flutter test app
cd test_app && mobile-use run -- -d <device_id>

# Android native
mobile-use -d <device_id> connect --package com.example.mobileuse.test_app

# iOS native
mobile-use -d <UDID> connect-ios --team-id <TEAM_ID>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liuqijun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/liuqijun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
