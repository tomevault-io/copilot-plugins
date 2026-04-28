## claude-feishu-channel

> <!-- Generated: 2026-04-08 | Updated: 2026-04-08 -->

<!-- Generated: 2026-04-08 | Updated: 2026-04-08 -->

# Lodestar (夜航星)

## Purpose
A Feishu (Lark) channel plugin for Claude Code that bridges Feishu group chats to Claude Code sessions via a two-process architecture: a daemon that listens on WebSocket and an MCP server that polls JSONL message files. Code name is **Lodestar** (English) / **夜航星** (Chinese, from the 章北海传 theme song) — the daemon is the silent guide star that keeps sessions navigating through the dark.

## Key Files

| File | Description |
|------|-------------|
| `server.ts` | MCP server (stdio) — runs inside each Claude Code session, provides reply/react/edit/fetch tools, polls JSONL for inbound messages |
| `daemon.ts` | Standalone daemon — Feishu WebSocket listener, tmux session manager, control commands (hi/restart/kill/clear) |
| `email-worker.ts` | IMAP/SMTP email bridge — watches mailbox, queues tasks per project, spawns `claude -p`, replies with results |
| `project-lock.ts` | Process-based mutex — scans /proc to detect running `claude` processes per project directory |
| `package.json` | Project manifest — Bun runtime, Lark SDK, MCP SDK, mail libs |
| `test-event.ts` | Diagnostic utility for Feishu API connectivity testing |
| `email-config.example.json` | Template for email worker configuration |
| `.mcp.json` | MCP server configuration for Claude Code plugin system |
| `CLAUDE.md` | Project documentation and architecture guide |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `.claude-plugin/` | Plugin manifest for Claude Code (see `.claude-plugin/AGENTS.md`) |
| `skills/` | User-invocable skill definitions (see `skills/AGENTS.md`) |
| `hooks/` | Event-driven notification scripts (see `hooks/AGENTS.md`) |

## For AI Agents

### Working In This Directory
- Runtime is **Bun**, not Node.js — use `bun run` for all scripts
- Group name = tmux session name = `~/` subdirectory name — this binding convention is load-bearing throughout
- The plugin is registered as `feishu` in the `feishu-channel` marketplace — no disguise needed
- State lives in `~/.claude/channels/feishu/`, not in this repo
- Credentials are in `~/.claude/channels/feishu/.env` — never commit them

### Testing Requirements
- `bun test-event.ts` — verify Feishu API connectivity and WebSocket events (30s diagnostic)
- Manual testing: send "hi" in a Feishu group to trigger session creation

### Common Patterns
- Async Feishu API calls via `@larksuiteoapi/node-sdk` Client
- JSONL files as IPC between daemon and server (no shared WebSocket)
- Interactive cards for permission relay (approve/deny buttons)
- Process detection via `/proc` scanning for mutual exclusion

## Dependencies

### External
- `@larksuiteoapi/node-sdk` — Feishu REST API + WebSocket client
- `@modelcontextprotocol/sdk` — MCP stdio server framework
- `zod` — Schema validation for MCP handlers
- `imapflow` — IMAP IDLE listener
- `nodemailer` — SMTP sender
- `mailparser` — Email body extraction

### System
- Bun (JS runtime)
- tmux (session management)
- Python 3 (notification hooks)

<!-- MANUAL: -->

---
> Source: [leviyuan/claude-feishu-channel](https://github.com/leviyuan/claude-feishu-channel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
