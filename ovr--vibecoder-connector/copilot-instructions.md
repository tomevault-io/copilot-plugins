## vibecoder-connector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin that delivers haptic feedback through Buttplug-compatible devices (via Intiface Central) during coding sessions. When Claude needs attention, finishes a task, or starts a session, connected devices vibrate with distinct patterns.

## Prerequisites

- **Intiface Central** must be running with its WebSocket server on `ws://127.0.0.1:12345` (default)
- At least one vibrating device paired in Intiface Central
- Node.js (ESM — the project uses `"type": "module"`)

## Commands

```bash
npm install                                          # install dependencies
node scripts/vibrate.js --pattern=<name>             # trigger a pattern manually (hello, attention, complete)
```

No build step, no tests, no linter configured.

## Architecture

This is a Claude Code plugin, not a standalone app. The entry point is `.claude-plugin/plugin.json`, which registers hooks defined in `hooks/hooks.json`.

**Hook flow:**
- `SessionStart` (startup/resume) -> fires `hello` pattern (double tap)
- `Notification` (permission_prompt/idle_prompt/elicitation_dialog) -> fires `attention` pattern (slow wave)
- `Stop` -> fires `complete` pattern (celebratory burst)

Each hook runs `scripts/vibrate.js --pattern=<name>` asynchronously with a 10s timeout.

**`scripts/vibrate.js`** — the single runtime script. Connects to Intiface Central via the `buttplug` npm package's WebSocket connector, discovers vibrating devices, plays the requested pattern (sequence of intensity+duration steps), then disconnects. Exits with code 0 on all errors to avoid blocking Claude Code.

**`config/patterns.json`** — defines vibration patterns as arrays of `{intensity, ms}` steps, plus `intifaceUrl` and `intensityMultiplier` (0-1 global scaling). Add new patterns here; they're immediately available via `--pattern=<name>`.

---
> Source: [ovr/vibecoder-connector](https://github.com/ovr/vibecoder-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
