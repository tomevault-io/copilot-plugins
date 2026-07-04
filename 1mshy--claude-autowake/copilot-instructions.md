## claude-autowake

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Claude Autowake is a macOS tool that pre-starts Claude Code's 5-hour rolling usage window by sending a minimal ping (`"hi"` to Haiku) on a schedule via launchd. This ensures the usage window resets before the user's workday begins.

## Architecture

Four shell scripts, no build system:

- **config.sh** — All user-configurable settings (ping times, model, weekday-only flag, caffeinate duration). Sourced by other scripts.
- **autowake.sh** — The ping runner. Sends one message to Claude CLI (`claude --print --model haiku -p "hi"`). Has lockfile protection, retry logic (2 attempts, 60s gap), and log pruning.
- **install.sh** — Generates two launchd plists (`com.autowake.ping` for scheduled pings, `com.autowake.caffeinate` to keep the Mac awake), loads them via `launchctl bootstrap`, and sets a `pmset repeat wakeorpoweron` schedule. Validates that ping times are >= 5h apart.
- **uninstall.sh** — Reverses install: unloads agents, removes plists, cancels pmset, optionally deletes logs.

## Key Commands

```bash
./install.sh       # Install launchd agents + pmset wake (requires sudo for pmset)
./uninstall.sh     # Remove everything
./autowake.sh      # Manual test ping
./status.sh        # Check if everything is loaded and show last ping result
```

## Runtime Artifacts

- Logs: `~/.claude-autowake/logs/ping_*.log` (auto-pruned to 30 files)
- Lockfile: `/tmp/claude-autowake.lock`
- LaunchAgents: `~/Library/LaunchAgents/com.autowake.{ping,caffeinate}.plist`

## Platform

macOS only — depends on launchd, pmset, and caffeinate.

---
> Source: [1mshy/claude-autowake](https://github.com/1mshy/claude-autowake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
