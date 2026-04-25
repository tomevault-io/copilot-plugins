## claw-monitor

> Context for AI coding agents working on this codebase.

# AGENTS.md — Claw-Monitor Developer Guide

Context for AI coding agents working on this codebase.

## Project Overview

Terminal dashboard (TUI) for monitoring OpenClaw agents, cron jobs, Docker containers, and system resources. Raw ANSI rendering + TypeScript.

## Tech Stack

- **Runtime**: Node.js 18+ (ESM modules)
- **UI**: Raw ANSI escape codes — alternate screen buffer, cursor-home overwrite, per-line erase
- **Language**: TypeScript (strict mode, ES2020 target, NodeNext modules)
- **Build**: `npm run build` → `tsc` → outputs to `dist/`
- **Entry point**: `src/index.ts` → `dist/index.js`
- **Run**: `npm start` or `./bin/claw-monitor.js`

## Project Structure

```
src/
├── index.ts                   # Entry point, alternate screen, keyboard input, render loop
├── data.ts                    # Centralised data collection with interval-based polling
├── render.ts                  # ANSI frame renderer — all dashboard sections
├── hooks/
│   ├── useSubAgents.ts        # Loads OpenClaw session directory + JSONL logs
│   ├── useCodingAgents.ts     # Detects coding agent processes via `ps aux`
│   ├── useCronJobs.ts         # Loads `openclaw cron list --json`
│   ├── useSystemCron.ts       # Loads `crontab -l`
│   └── useSysStats.ts         # Collects CPU/MEM/DISK/GPU/Docker stats
└── utils/
    ├── ansi.ts                # ANSI escape code helpers (colors, bold, dim, spinner)
    ├── config.ts              # All tunables with env-var overrides
    ├── cronUtils.ts           # Shared: fit(), relativeTime(), cronToHuman(), nextCronRun()
    └── parseSession.ts        # JSONL session log parser
bin/
├── claw-monitor.js            # CLI entry point (shebang wrapper)
├── cc-attach                  # tmux attach script for Claude Code
├── copilot-attach             # tmux attach script for Copilot CLI
└── codex-attach               # tmux attach script for Codex
```

## Architecture

### Rendering

Uses raw ANSI escape codes with alternate screen buffer. `index.ts` runs a `setInterval` render loop that calls `renderFrame()` from `render.ts`. Each frame overwrites in-place with cursor-home + per-line erase (`\x1b[K`) to prevent ghosting. Spinner characters are stripped for stable-frame comparison to avoid needless redraws.

### Data Flow

`DataCollector` in `data.ts` polls each data source on independent intervals:

| Source | Loader Function | Default Interval |
|--------|----------------|-----------------|
| Sub-agents | `loadSessionsData()` + file system | 500ms |
| Coding agents | `detectAgents()` via `ps aux` | 5s |
| Cron jobs | `loadCronJobs()` via `openclaw cron list --json` | 15s |
| System cron | `loadSystemCron()` via `crontab -l` | 60s |
| System stats | `collectStats()` via `top`, `os.*`, `df`, `nvidia-smi`, `docker ps`, `systemctl`, `kubectl` | 10s |

All intervals are configurable via environment variables (see `src/utils/config.ts`).

### Layout

`render.ts` composes sections top-to-bottom inside a box-drawing border:

1. Coding Agents (if any detected)
2. Sub-Agents (with attach commands and detach hint)
3. OpenClaw Cron Jobs (if available)
4. System Cron Jobs (if available)
5. System Stats — two-column: resource bars (left) + Docker containers (right)
6. Footer — agent count summary
7. Help hint — keyboard shortcuts (contextual: ↑↓/Enter only shown when agents exist)

Width is responsive (60–120 columns) via terminal resize events in `index.ts`. When content exceeds terminal height, PgUp/PgDn/Home/End provide internal scrolling. The alternate screen buffer preserves the user's original scrollback on exit.

## Key Conventions

- **Box-drawing borders**: All sections use manual `│`, `┌`, `└`, `├`, `┤`, `─` characters with padding to `boxWidth`. Components are responsible for their own left/right borders.
- **Emoji width**: Emoji like 🐳 are 2 visual columns but 1 JS char. Account for this in padding math.
- **`fit()` helper**: Truncates or pads strings to exact column width. Use it for table columns.
- **Warnings**: Hooks return a `warning` string when commands fail (e.g., `openclaw` not found). These render as yellow `⚠` lines inside the relevant section.
- **Anti-flicker**: Spinner chars are stripped for frame comparison; identical frames are not rewritten.
- **Config via env vars**: Every tunable (poll intervals, thresholds, paths) lives in `src/utils/config.ts` with an `envInt`/`envStr` helper.

## Build & Run

```bash
npm run build      # TypeScript compile
npm start          # Run the dashboard
npm run dev        # Watch mode (tsc --watch)
npm run clean      # Remove dist/
```

## Common Tasks

**Add a new dashboard section:**
1. Create a loader function in `src/hooks/` that collects and returns typed data
2. Add the loader + poll interval to `DataCollector` in `src/data.ts`
3. Add a render function in `src/render.ts` within the frame layout

**Add a new system stat:**
1. Add detection + parsing in `useSysStats.ts` (use `commandExists()` for optional tools)
2. Add the data to the `SysStats` interface
3. Render in `render.ts`

**Change poll intervals:**
Set environment variables: `POLL_AGENTS=1000 POLL_STATS=5000 claw-monitor`

---
> Source: [DanWahlin/claw-monitor](https://github.com/DanWahlin/claw-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
