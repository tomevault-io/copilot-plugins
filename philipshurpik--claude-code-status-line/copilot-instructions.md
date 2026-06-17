## claude-code-status-line

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A color-coded status line for Claude Code CLI. Single Node.js script (`status-line.js`) that runs as a StatusLine hook — receives live `context_window` and `rate_limits` data on every streaming update and displays a color-coded status bar in the terminal.

Fully self-contained, no dependencies.

## Architecture

```
Claude Code CLI
    └─► StatusLine hook (status-line.js)   fires on every token stream update
            prints: colored status bar → stdout → Claude Code displays it
```

**Token thresholds** (defined in status-line.js):
- `AUTOCOMPACT_BUFFER_TOKENS` = 33K — Claude Code reserves this for autocompact; subtracted from raw window to get effective window size
- `WARN_TOKENS` = 60K — status turns yellow/warning
- `COMPACT_TOKENS` = 80K — status turns orange/consider compacting

## Commands

```bash
node --test tests/test_status_line.js   # run tests
bash install.sh                         # install
```

Tests use `node:test` (no deps), run the hook via `execFileSync`. 
CI: GitHub Actions on push/PR to main, Node 20.

---
> Source: [philipshurpik/claude-code-status-line](https://github.com/philipshurpik/claude-code-status-line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
