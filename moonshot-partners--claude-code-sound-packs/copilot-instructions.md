## claude-code-sound-packs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sound packs for Claude Code hooks. Each pack maps WAV sound files to 8 Claude Code hook events (SessionStart, UserPromptSubmit, Notification, Stop, SubagentStart, SubagentStop, TaskCompleted, SessionEnd). Activation writes hook entries into `~/.claude/settings.json` that play sounds via `afplay`.

macOS only (uses `afplay`). Requires `jq`, `ffmpeg`, and `curl` for installation.

## Commands

```bash
./install.sh <pack-name|all>   # Download sources, convert to WAV, copy to ~/.claude/hooks/sounds/
./activate.sh <pack-name>      # Merge sound hooks into ~/.claude/settings.json
./deactivate.sh                # Remove sound hooks from settings.json
./preview.sh <pack-name>       # Play all sounds in a pack via afplay
```

## Architecture

All scripts source `lib/common.sh` which defines shared constants (paths, colors, event names) and helpers (pack validation, dependency checking, settings management).

**Key paths managed by the scripts:**
- `~/.claude/settings.json` — Claude Code settings file where hooks are written
- `~/.claude/hooks/sounds/<pack-name>/` — installed WAV files
- `~/.claude/hooks/sounds/.active-pack.json` — manifest tracking which events are managed by the active pack (used for clean deactivation)

**Pack structure** — each pack lives in `packs/<name>/` with:
- `pack.json` — metadata + event-to-sound mappings (keys are event names, values have `file` and `label`)
- `sources.json` — download URLs and conversion format (the `files` array maps URLs to target WAV paths)
- `sounds/` — pre-built WAV files committed to the repo (install.sh prefers these over re-downloading)
- `raw/` — original source audio files (gitignored)

**Async behavior:** `activate.sh` marks frequent events (UserPromptSubmit, Notification, SubagentStart, SubagentStop, TaskCompleted) with `async: true` so sound playback doesn't block Claude.

**Settings safety:** `activate.sh` and `deactivate.sh` only touch hook events listed in the pack's manifest. Non-sound hooks and all other settings are preserved.

---
> Source: [moonshot-partners/claude-code-sound-packs](https://github.com/moonshot-partners/claude-code-sound-packs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
