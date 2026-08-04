## logiops-rs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

logiops-rs is a Rust rewrite of [logiops](https://github.com/PixlOne/logiops), a daemon for configuring Logitech mice and keyboards on Linux via the HID++ protocol.

**Current status**: Phase 5 complete (feature parity) + GUI. Hi-res scrolling, thumb wheel, and on-board profiles implemented. D-Bus interface, systemd service, and egui GUI ready. See `STATUS.md` for dev log and `docs/internal/` for detailed implementation plans.

## Build Commands

```bash
cargo build --release                          # Build all crates
cargo build --package logiops-daemon           # Build daemon
cargo build --package logiops-gui              # Build GUI
sudo target/release/logiops                    # Run daemon (needs root for /dev/hidraw*)
cargo run -p logiops-gui                       # Run GUI (connects to daemon via D-Bus)
```

## Testing

```bash
cargo test --lib --all-features                # All unit tests
cargo test --package logiops-core test_name    # Single test by name
RUST_LOG=debug sudo target/release/logiops     # Run with debug logging
```

### D-Bus Testing

```bash
busctl list | grep logiops                     # Check service registered
busctl introspect org.logiops /org/logiops/Manager  # View interface
busctl call org.logiops /org/logiops/Manager org.logiops.Manager1 ListDevices
busctl call org.logiops /org/logiops/Manager org.logiops.Manager1 ReloadConfig
```

## Linting

```bash
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings      # Pedantic mode enabled in workspace
```

## Architecture

**Actor-based async design** using tokio. Each Logitech device runs as an independent actor task communicating via mpsc channels.

### Workspace Crates

- **hidpp-transport**: HID transport layer wrapping hidapi. Provides `HidapiChannel` for async-safe HID communication and device enumeration.
- **logiops-core**: HID++ protocol implementation. Contains feature implementations (IRoot, IFeatureSet, DPI, SmartShift, Battery) and the `HidppDevice` abstraction that auto-discovers features.
- **logiops-daemon**: Main binary with actor system, device registry, udev hotplug monitor, button remapping via uinput, configuration loading (figment), and logging (tracing).
- **logiops-gui**: egui-based GUI for device configuration. Connects to daemon via D-Bus as a regular user. Supports DPI, SmartShift, hi-res scroll, thumb wheel, and battery status.

### Key Components (logiops-daemon)

- **DeviceActor** (`actor.rs`): Owns HID channel, processes commands via mpsc. One actor per connected device. Polls for button events when configured.
- **DeviceHandle**: Cloneable handle for sending commands to an actor (get/set DPI, battery, SmartShift, buttons).
- **DeviceRegistry** (`registry.rs`): Thread-safe registry of active devices. Handles add/remove with proper actor lifecycle.
- **UdevMonitor** (`udev_monitor.rs`): Runs on dedicated thread (udev types not Send), sends hotplug events via tokio mpsc.
- **ActionExecutor** (`action.rs`): Handles button action execution (Keypress, CycleDpi, ToggleSmartShift).
- **LogiopsVirtualDevice** (`virtual_device.rs`): uinput wrapper for emitting keyboard/mouse events.
- **D-Bus Manager** (`dbus/manager.rs`): org.logiops.Manager1 interface for remote device management.
- **KWin Integration** (`kwin.rs`): Sets scroll factor via KWin D-Bus on device connect (KDE Plasma Wayland).
- **systemd** (`systemd.rs`): sd-notify integration for Type=notify service lifecycle.

### GUI Components (logiops-gui)

- **App** (`app.rs`): Main eframe application with async D-Bus bridge via Command/Response channels.
- **DbusWorker** (`dbus.rs`): Tokio task handling D-Bus communication; uses zbus proxy for `org.logiops.Manager1`.
- **State** (`state.rs`): UI state structs (`AppState`, `DeviceState`, `DpiState`, `SmartShiftState`, etc.).
- **Components** (`components/`): Modular UI components for device list, DPI slider, SmartShift controls, hi-res scroll, thumb wheel, battery display.

GUI runs as regular user (no root needed) - connects to daemon via system D-Bus. Optional `--features mcp` enables egui-mcp-bridge for E2E testing.

### HID++ Feature Implementations (logiops-core)

| Feature | ID | File |
|---------|-----|------|
| IRoot | 0x0000 | `features/root.rs` |
| IFeatureSet | 0x0001 | `features/feature_set.rs` |
| DeviceName | 0x0005 | `features/device_name.rs` |
| Unified Battery | 0x1000 | `features/battery.rs` |
| Battery Status | 0x1001 | `features/battery.rs` |
| Reprog Controls | 0x1B04 | `features/reprog_controls.rs` |
| SmartShift | 0x2110 | `features/smartshift.rs` |
| HiRes Scrolling | 0x2121 | `features/hires_scrolling.rs` |
| Thumb Wheel | 0x2150 | `features/thumb_wheel.rs` |
| Adjustable DPI | 0x2201 | `features/adjustable_dpi.rs` |
| On-board Profiles | 0x8100 | `features/onboard_profiles.rs` |

### Configuration Layering

figment loads in order: defaults → `/etc/logiops/config.toml` → `~/.config/logiops/config.toml` → `LOGIOPS_*` env vars

## HID++ Protocol Notes

- **Logitech VID**: 0x046d
- **HID++ interface**: USB uses interface 2; Bluetooth uses interface -1
- **Report sizes**: Short = 7 bytes (report ID 0x10), Long = 20 bytes (0x11), Very Long = 64 bytes (0x12)
- **Device index**: Direct USB/Bluetooth uses 0xFF; Unifying/Bolt receivers use 1-6 for paired devices
- **Feature discovery**: Query IRoot (always at index 0) to find feature indices; indices vary per device/firmware
- **Software ID**: Include in byte 3 of requests (low nibble) to match responses

## KWin Integration (KDE Plasma Wayland)

KDE Plasma on Wayland has a bug where mouse scroll settings don't apply to hotplugged/Bluetooth devices. logiops-rs works around this by calling KWin's D-Bus interface to set the scroll factor when a device connects.

```toml
[[devices]]
name = "MX Master 3S"
kwin_scroll_factor = 1.8  # Sets scroll speed via KWin D-Bus
```

This feature:
- Only activates on KDE Plasma (session D-Bus required)
- Retries automatically if KWin hasn't registered the device yet
- Works for both USB and Bluetooth devices
- Eliminates the need for separate scripts or systemd workarounds

## Known Limitations

**Bluetooth HID++ conflict**: On Linux, the kernel's `logitech-hidpp-device` driver intercepts HID++ responses from Bluetooth devices, preventing configuration. USB-connected devices (including via Unifying/Bolt receiver) work normally. See [docs/bluetooth-limitations.md](./docs/bluetooth-limitations.md) for full technical analysis.

## Installation

```bash
# Install binary and system files
sudo cp target/release/logiops /usr/bin/
sudo cp data/dbus/org.logiops.conf /usr/share/dbus-1/system.d/
sudo cp data/systemd/logiops.service /etc/systemd/system/
sudo cp data/udev/99-logiops.rules /etc/udev/rules.d/

# Reload and start
sudo systemctl daemon-reload
sudo udevadm control --reload-rules
sudo systemctl enable --now logiops
```

## Permissions

Daemon needs access to:
- `/dev/hidraw*` - HID communication (run as root or udev rule for group `plugdev`)
- `/dev/uinput` - Virtual input devices for button remapping (run as root or udev rule for group `input`)

## MSRV

Rust 1.82.0 (required by indexmap dependency)

---
> Source: [aydiler/logiops-rs](https://github.com/aydiler/logiops-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
