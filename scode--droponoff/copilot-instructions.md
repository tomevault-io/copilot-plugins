## droponoff

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`droponoff` is a macOS-specific CLI tool that provides a reversible kill switch for Dropbox. It allows users to completely disable Dropbox (processes, LaunchAgents, File Provider extensions) to safely manipulate internal Dropbox state files, then restore it to normal operation.

## Build and Run

```bash
# Build the project
cargo build

# Build release version
cargo build --release

# Run the tool
cargo run -- <command>

# Or after building
./target/debug/droponoff <command>
```

## Commands

- `droponoff off` - Disable Dropbox completely
- `droponoff on` - Restore Dropbox to normal operation
- `droponoff status` - Show current Dropbox state (read-only)

## Architecture

### Module Organization

The codebase follows a functional module structure where each module handles a specific aspect of Dropbox management:

- **discovery.rs** - Locates Dropbox components (app bundle, LaunchAgent paths, extension bundle IDs)
- **processes.rs** - Process enumeration and graceful shutdown via AppleScript
- **launchagent.rs** - LaunchAgent load/unload and enable/disable via file renaming
- **extensions.rs** - PlugInKit extension management (enable/disable via `pluginkit` command)
- **finder.rs** - Finder restart to refresh extension state
- **status.rs** - Aggregates and displays current state
- **sudo.rs** - Helper for privilege escalation (currently unused)

### Key Design Principles

**Graceful Shutdown Only**: The tool only uses non-forceful means to stop processes. It sends an AppleScript quit request to Dropbox and then waits for processes to terminate naturally. No `pkill` or force-kill operations are used.

**Reversibility**: LaunchAgent plists are renamed (`.plist` ↔ `.plist.disabled`) rather than deleted, ensuring clean restoration.

**Order of Operations for OFF**:
1. Request Dropbox to quit gracefully (AppleScript)
2. Disable LaunchAgent (unload + rename)
3. Disable extensions (via `pluginkit -e ignore`)
4. Restart Finder
5. **Wait for all processes to stop** (this happens after disabling agents/extensions so all related processes are caught)
6. Verify state

**Order of Operations for ON**:
1. Restore LaunchAgent (rename + load)
2. Enable extensions (via `pluginkit -e use`)
3. Restart Finder
4. Launch Dropbox app
5. Wait for processes to start
6. Verify state

### macOS-Specific Components

**PlugInKit Extensions**: Three bundle IDs are managed:
- `com.getdropbox.dropbox.fileprovider` (File Provider)
- `com.getdropbox.dropbox.TransferExtension`
- `com.getdropbox.dropbox.garcon`

**LaunchAgent**: `~/Library/LaunchAgents/com.dropbox.DropboxMacUpdate.agent.plist` is managed via modern `launchctl bootstrap/bootout` commands with gui domain.

**Process Detection**: Uses `pgrep -l -u $USER -f Dropbox` to find all Dropbox-related processes for the current user.

## Logging

The tool uses the `tracing` library for all output. The default configuration (set in `main.rs`) produces clean output without timestamps, log levels, or targets, mimicking simple println-style output. Users can customize via `RUST_LOG` environment variable if needed.

## Platform Guard

The entire crate is guarded with `#[cfg(not(target_os = "macos"))]` compile error to prevent compilation on non-macOS platforms.

## Implementation Notes

- The `sudo.rs` module exists but is currently unused. All operations run at user level.
- Error handling uses `anyhow::Result` throughout with strict error propagation.
- The tool is idempotent - running `off` or `on` multiple times is safe.
- See `REQUIREMENTS.md` for the full behavioral specification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
