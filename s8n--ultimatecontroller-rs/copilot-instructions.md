## ultimatecontroller-rs

> ultimatecontroller is a Rust CLI + GUI tool for configuring 8BitDo Ultimate controllers on Linux. The HID protocol was reverse-engineered from the official Windows (`8BitDoAdvance.dll`) and Android (`libadvance-lib.so`) applications, and verified with USB/BLE traffic captures.

# CLAUDE.md

## Project Overview

ultimatecontroller is a Rust CLI + GUI tool for configuring 8BitDo Ultimate controllers on Linux. The HID protocol was reverse-engineered from the official Windows (`8BitDoAdvance.dll`) and Android (`libadvance-lib.so`) applications, and verified with USB/BLE traffic captures.

## Build Commands

```bash
cargo build                              # CLI only
cargo build --features gui               # CLI + GUI
cargo build --features gui,ble           # CLI + GUI + BLE
cargo build --features gui,ble,calibration  # All features
cargo test --lib --features gui          # Run tests (skip doctests)
```

All feature combinations must compile cleanly with zero warnings.

## Architecture

- **Single crate, two binaries**: `ultimatecontroller` (CLI) and `ultimatecontroller-gui` (GUI, requires `gui` feature)
- **Protocol layer** (`src/protocol/`): Pure protocol implementation. `Transport` trait in `hid.rs` abstracts USB HID vs BLE. Types in `structs.rs`, packet building in `hid.rs`, commands in `commands.rs`.
- **Device layer** (`src/devices/`): Device detection, USB operations (`usb.rs` shared between CLI and GUI), BLE transport (`ble.rs` with `BleTransport` sync wrapper).
- **GUI layer** (`src/gui/`): egui/eframe app. All device I/O runs on a background thread (`device_thread.rs`) communicating via mpsc channels. UI thread never blocks.

## Key Types

- `Transport` trait — abstracts USB HID vs BLE packet I/O. `HidDevice` and `BleTransport` implement it.
- `ControllerConfig` — 1652-byte config struct with 3 profile slots. Has `PartialEq` for dirty tracking.
- `Command` / `Status` — mpsc message enums between GUI and device thread.
- `DeviceHandle` — enum holding either `Hid(HidDevice, pid)` or `Ble(BleDevice, Runtime)`.
- `BleTransport` — sync wrapper around BleDevice + tokio Runtime, implements `Transport`.

## Protocol Details

USB uses 64-byte HID reports with report ID `0x81`. BLE uses the same protocol but different framing:
- **USB write**: `[0x81, length, 0x04, cmd, ...]` (64 bytes, padded)
- **BLE write**: `[0x04, cmd, ...]` (payload only, no report_id, no length byte)
- **USB response**: `[status(02), type(04), cmd, sub, ...]`
- **BLE response**: `[type(04), 0x00, cmd, ...]` (no status byte, extra 0x00)
- BLE does NOT need `CMD_REPORT_STATE` (enter config mode) — go straight to read/write
- Write responses use `cmd=CMD_READ_RESPONSE(4), sub=CMD_WRITE(1)` (not `cmd=1`)

Known commands:
- Config read/write: transfer packets (0x04 type), 45 bytes per chunk
- Version: direct command 0x00 0x21
- Vibration: simple command cmd=0x08 (pcap-verified)
- Calibration: direct commands 0x00 0x36/0x38 (behind `calibration` feature flag, untested)

## Documentation

**`docs/protocol.md` must be kept up to date** when protocol behavior is discovered or corrected. It is the primary reference for the reverse-engineered protocol and has been verified against USB and BLE traffic captures. Update it whenever:
- A new command is discovered or an existing one is corrected
- BLE vs USB differences are found
- Packet format details change
- New device PIDs or firmware types are identified

## Platform-Specific Code

Linux-specific code (xpad rmmod, sysfs unbind, kernel driver detach) is gated behind `#[cfg(target_os = "linux")]` with no-op stubs for other platforms. All in `src/devices/usb.rs` and `src/gui/device_thread.rs`.

## Feature Flags

- `gui` — enables eframe/egui GUI binary
- `ble` — enables btleplug BLE transport (working — tested with Ultimate BT)
- `calibration` — enables stick calibration commands (untested, potentially risky)

All feature-gated code uses `#[cfg(feature = "...")]`. GUI must compile with and without BLE/calibration.

## Conventions

- All numbers in the protocol are **little-endian**
- Button mappings use **bitmask values** (not indices) — see `constants.rs`
- P1/P2 paddles can be remapped FROM but not TO (not in `REMAP_TARGETS`)
- Config has 3 slots indexed 0-2; `cur_slot` determines active profile on controller
- Controller stores ONE button mapping per slot (shared across gamepad modes). Per-mode mappings are a Windows app client-side feature.
- Logging uses the `log` crate: `trace` for packet hex, `debug` for flow, `info` for events, `warn` for retries, `error` for failures
- GUI defaults to `info` log level; CLI defaults to none. Override with `RUST_LOG=debug`.
- Mode switch retries up to 3 times (controller often needs multiple attempts)

## RE Artifacts

Reverse engineering materials live in `re/` (gitignored). Decompiled sources, APKs, pcap captures, firmware updater. The `scripts/download_firmware.py` script fetches firmware from 8BitDo's server (firmware is Jieli-encrypted, not flashable by this tool).

## Testing Notes

- No controller hardware required for compilation
- Unit tests exist for CRC; most functionality requires a physical device
- The `hidapi` and `rusb` crates need system libraries (`libhidapi-dev`, `libusb-1.0-0-dev` on Debian/Ubuntu, `hidapi` and `libusb` on Arch)
- BLE tested with Ultimate BT (PID 0x6007, BLE name "80NA", characteristic FF13)
- USB tested with Ultimate BT in config mode (PID 0x6007)
- AVOID sending cmd=0x0D over HID — it resets/disconnects the controller

---
> Source: [s8n/ultimatecontroller-rs](https://github.com/s8n/ultimatecontroller-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
