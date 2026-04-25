## feishuchannel-for-claudecode

> A Feishu (Lark) channel plugin for Claude Code using the MCP Channel protocol. Provides WebSocket-based messaging between Feishu and Claude Code instances, with an optional router for multi-group routing.

# Feishu Channel for Claude Code

## Project Overview

A Feishu (Lark) channel plugin for Claude Code using the MCP Channel protocol. Provides WebSocket-based messaging between Feishu and Claude Code instances, with an optional router for multi-group routing.

## Architecture

- **server.ts** — MCP server plugin. Runs in three modes:
  - **Worker mode**: connects to router via Unix socket (auto-starts router if needed)
  - **Channel mode**: direct Feishu WebSocket connection (fallback if router fails to start)
  - **Passive mode**: no remote connection (non-channel Claude instances)
- **router.ts** — Central hub. Single Feishu WebSocket, routes messages to workers by workdir. Auto-spawned by the first worker that starts
- **bin/claude-feishu** — Shortcut for `claude --dangerously-load-development-channels plugin:feishu@feishu-local`

## Key Design Decisions

- Process tree walking (`ps -o pid=,ppid=,args= -ax`) detects whether the parent Claude Code was launched with `--channels feishu`
- `lsof -a -p <pid> -d cwd -Fn` resolves the Claude ancestor's actual working directory (plugin runs from cache dir, not project dir)
- ppid polling every 2s for orphan detection (Bun doesn't fire stdin end/close on broken Unix domain sockets)
- Router-worker IPC uses newline-delimited JSON over Unix socket
- Message routing: `chat_id → group.workdir (access.json) → registered worker (by cwd)`

## State Directory

`~/.claude/channels/feishu/` — credentials, access control, debug logs, router socket.

## Tech Stack

- **Runtime**: Bun
- **SDK**: `@larksuiteoapi/node-sdk` (Feishu API + WebSocket), `@modelcontextprotocol/sdk` (MCP)
- **Tests**: `bun:test`

## Commands

```bash
bun test              # Run tests
bun run start         # Start MCP server (called by Claude Code)
bun run router        # Start multi-group router
claude-feishu         # Launch Claude Code with Feishu channel
```

## Testing

Tests are in `server.test.ts`. They extract pure functions from server.ts/router.ts logic and test them without side effects. Run with `bun test`.

## Plugin Installation

```bash
claude plugin marketplace add /path/to/feishuchannel
claude plugin install feishu@feishu-local
```

Plugin cache: `~/.claude/plugins/cache/feishu-local/feishu/0.1.0/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phxwang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
