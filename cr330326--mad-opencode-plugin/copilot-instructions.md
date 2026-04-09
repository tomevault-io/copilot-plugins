## mad-opencode-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MAD (Multi-Agent Dashboard) is a monitoring and visualization platform for AI Agent collaboration scenarios. It captures events from OpenCode CLI and syncs them to a central dashboard for real-time visualization of multi-agent trees, session management, and dependency graphs.

## Repository Structure

```
Multi-Agent-Plugin/
├── README.md              # Project overview (Chinese)
├── DESIGN-SPEC.md         # Detailed system design specification
└── code/
    └── mad-opencode-plugin/   # OpenCode plugin (npm package)
        ├── src/
        │   ├── index.ts       # Plugin entry point, hooks registration
        │   ├── types.ts       # TypeScript type definitions
        │   ├── queue.ts       # EventQueue with offline retry
        │   └── logger.ts      # File-based logger (bypasses TUI suppression)
        ├── dist/              # Compiled output
        ├── Makefile           # Build commands
        └── package.json
```

## Build Commands

```bash
cd code/mad-opencode-plugin

# Build the plugin
make build        # or: npm run build

# Development mode (watch for changes)
make dev          # or: npm run dev

# Clean build artifacts
make clean        # or: npm run clean

# Run tests
make test

# All commands
make help
```

## Architecture

The plugin uses the `@opencode-ai/plugin` SDK to register hooks that capture OpenCode events:

- **event hook**: Captures session create/update/compact events
- **chat.message hook**: Captures user/assistant messages with model info
- **tool.execute.after hook**: Captures tool executions (Read, Bash, task, etc.)

Events are pushed to an `EventQueue` that:
- Buffers up to 1000 events
- Sends to MAD Server via HTTP POST
- Retries with exponential backoff (5 attempts max, 1s-30s delay)
- Flushes every 5 seconds

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MAD_SERVER_URL` | `http://localhost:3000` | MAD Server endpoint |
| `MAD_API_KEY` | `dev-key` | API authentication key |
| `MAD_CLIENT_NAME` | `hostname()` | Client identifier |
| `MAD_DEBUG` | - | Set to `1` for debug logging |

## Local Development Setup

For local testing without npm publish:

```bash
cd code/mad-opencode-plugin
make setup   # Runs scripts/setup.sh

# Configure OpenCode to use local plugin
# Edit ~/.config/opencode/opencode.json:
# { "plugin": ["file:///Users/YOUR_USER/.cache/opencode/node_modules/@mad/opencode-plugin"] }
```

## Logs

Plugin logs are written to `~/.config/opencode/log/mad-plugin.log` (bypasses TUI terminal suppression):

```bash
# View logs
cat ~/.config/opencode/log/mad-plugin.log

# Real-time monitoring
tail -f ~/.config/opencode/log/mad-plugin.log
```

## Key Implementation Details

- Plugin state is managed per-instance via `WeakMap<PluginInput, PluginState>` for proper isolation
- Heartbeat events are sent every 30 seconds with memory/uptime metrics
- Process exit handlers (SIGTERM, SIGINT, beforeExit) ensure graceful shutdown with final queue flush
- Timer unref() is called to allow Node.js to exit when only heartbeat/flush timers are active

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cr330326)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cr330326)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
