## notify-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

notify-me is a macOS notification system for daily reminders (water intake, breaks, etc.) delivered as native system alerts.

## Architecture

- **notify_me/**: Main Python package
  - `config.py`: Reminder configuration and schedule definitions
  - `notifier.py`: macOS notification delivery via `osascript`
  - `scheduler.py`: Determines which notifications to send based on current time
  - `cli.py`: Command-line interface
- **launchd/**: macOS launchd plist files for background scheduling

## Commands

```bash
# Install in development mode
uv pip install -e ".[dev]"

# Run notifications check (sends any due notifications)
uv run notify-me check

# List configured reminders
uv run notify-me list

# Test a specific notification
uv run notify-me test "Test message"

# Install/uninstall launchd service
uv run notify-me install
uv run notify-me uninstall
```

## Technical Notes

- Uses `osascript` for native macOS notifications (no external dependencies)
- launchd runs the check command periodically (every 15 minutes by default)
- Reminder state (last sent times) stored in `~/.local/share/notify-me/state.json`
- Config file at `~/.config/notify-me/config.toml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cmyui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
