## windows-screen-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
cargo build                        # debug build
cargo build --release              # optimized build → target/release/wsm.exe
cargo run --bin wsm -- save        # run save subcommand (outputs config.json)
cargo run --bin wsm -- load <file> # run load subcommand
cargo check                        # fast type-check without linking
```

There are no tests yet. The binary name is `wsm` (not the crate name `win-screen-manager`).

## Architecture

Single-file Rust program (`src/main.rs`) targeting Windows only via the `windows` crate.

**Two subcommands (via `clap`):**
- `save [output.json]` — enumerates active monitors with `EnumDisplayDevicesW` + `EnumDisplaySettingsExW`, serializes to JSON
- `load <config.json>` — deserializes JSON, stages each monitor with `ChangeDisplaySettingsExW(CDS_UPDATEREGISTRY | CDS_NORESET)`, then commits all at once with a final `ChangeDisplaySettingsExW(null, null, CDS_TYPE(0))`

**Key types:**
- `MonitorConfig` — serde-serializable struct representing one display; `orientation` stores the raw `u32` from `DEVMODE_DISPLAY_ORIENTATION`
- All Win32 calls are `unsafe`; anonymous union fields in `DEVMODEW` are accessed via `devmode.Anonymous1.Anonymous2.{dmPosition, dmDisplayOrientation}`

**Windows crate notes:**
- `DISPLAY_DEVICE_ATTACHED_TO_DESKTOP` and `DISPLAY_DEVICE_PRIMARY_DEVICE` are plain `u32` constants — no `.0` unwrap needed
- `dmDisplayOrientation` is typed as `DEVMODE_DISPLAY_ORIENTATION(u32)` — use `.0` to extract and wrap when setting
- `EnumDisplaySettingsExW` last arg must be `ENUM_DISPLAY_SETTINGS_FLAGS(0)`, not a bare `0`
- Final commit call uses `CDS_TYPE(0)` for the flags parameter

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OsirisFrik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OsirisFrik)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
