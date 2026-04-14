## claude-channel-mux

> Multiplexes Claude Code channel sessions through a single Discord bot connection.

# claude-channel-mux

Multiplexes Claude Code channel sessions through a single Discord bot connection.

## Architecture

```
[Discord Bot]
       |
   [Daemon Process]  <- single bot connection, IPC server
       | Unix socket (JSON Lines)
   +---+---+
   |   |   |
 [A] [B] [C]  <- MCP server plugins, one per Claude Code session
```

- **Daemon**: Long-running process. Connects to Discord, listens on a Unix socket, routes messages between Discord and plugins.
- **Plugin**: Short-lived MCP server spawned by Claude Code. Connects to the daemon's Unix socket, registers channel claims, exposes MCP tools.

## Dev Setup

```bash
pnpm install
pnpm run check       # lint + build + typecheck + test + smoke (full pre-PR check)
pnpm run dev         # start daemon with watch mode
```

## Commands

| Command | What it does |
|---|---|
| `pnpm run check` | Run all checks (lint, build, typecheck, test, smoke) |
| `pnpm run lint` | Biome lint check |
| `pnpm run lint:fix` | Auto-fix lint issues |
| `pnpm run format` | Format code with Biome |
| `pnpm run build` | Build all packages |
| `pnpm run typecheck` | Type check (builds core first for type resolution) |
| `pnpm run test` | Unit tests per package |
| `pnpm run test:smoke` | Integration smoke tests (build + pack + verify) |
| `pnpm run dev` | Start daemon with watch mode |

## Monorepo Structure

pnpm workspace monorepo with three packages:

```
packages/
  core/       @claude-channel-mux/core      IPC, types, gate, config
  discord/    @claude-channel-mux/discord    Discord adapter, daemon, plugin
  cli/        @claude-channel-mux/cli        CLI (daemon, session)
```

### Dependency graph

```
cli -> core, discord
discord -> core
core -> (no internal deps)
```

### packages/core
- `types.ts`: IPC protocol messages, PlatformAdapter interface, Access types
- `config.ts`: State directory paths (~/.claude/channels/channel-mux/), .env loader
- `ipc-server.ts`: Unix socket server, session registry, message routing
- `ipc-client.ts`: Unix socket client, request/response correlation with timeout
- `gate.ts`: Pure gate logic (evaluateGate) for access control decisions

### packages/discord
- `adapter.ts`: Discord.js PlatformAdapter (reply, react, edit, fetch, download)
- `access.ts`: Access file I/O, discord-specific gate wrapper, isMentioned, approval polling
- `utils.ts`: chunk(), assertSendable(), safeAttName()
- `daemon.ts`: Wires Discord adapter + IPC server, signal handling
- `plugin.ts`: MCP server with 5 tools, IPC-proxied to daemon

### packages/cli
- `cli.ts`: `channel-mux daemon` (start/stop/status/group) and `channel-mux session` (view/channels/dms)

## Coding Conventions

- **TypeScript strict**, ESM (`"type": "module"`)
- **Biome** for lint + format (single quotes, no semicolons, trailing commas)
- **kebab-case** for all files and folders
- **pnpm** workspaces with `workspace:*` protocol
- Keep imports from `@claude-channel-mux/core` (not relative cross-package paths)
- Pure logic in `packages/core`, platform-specific code in adapter packages
- Tests live in `packages/*/tests/` (vitest)
- No em dash characters in code, docs, or issues

## Common Tasks

### Add a new MCP tool
1. Add tool schema in `packages/discord/src/plugin.ts` (ListToolsRequestSchema handler)
2. Add tool_call proxy in CallToolRequestSchema handler
3. Add tool type to `packages/core/src/types.ts` (ToolCallMsg.tool union)
4. Implement in `packages/discord/src/adapter.ts`
5. Wire in `packages/discord/src/daemon.ts` (ipc.onToolCall switch)

### Add a test
- Core tests: `packages/core/tests/`
- Discord tests: `packages/discord/tests/`
- Smoke tests: `tests/smoke/` (root level)
- Run: `pnpm run test` (unit) or `pnpm run test:smoke` (integration)

### Release
- `npx release-it <version>` (e.g. `npx release-it 0.1.0-alpha.2`)
- release-it bumps root version, sync-versions.ts copies to all packages
- CI publishes to npm and creates GitHub Release

## State Directory

All runtime state lives in `~/.claude/channels/channel-mux/`:
- `.env` -- Bot token (`DISCORD_BOT_TOKEN=...`)
- `access.json` -- Access control config
- `daemon.pid` / `daemon.sock` -- Daemon process files
- `inbox/` -- Downloaded attachments
- `approved/` -- Pairing approval files

## IPC Protocol

JSON Lines (newline-delimited JSON) over Unix domain socket at `~/.claude/channels/channel-mux/daemon.sock`. See `packages/core/src/types.ts` for all message type definitions.

## Key Design Decisions

- **Gate logic separated from adapters**: Core `evaluateGate()` operates on plain data. Discord adapter extracts fields from discord.js Message, resolves mentions, then delegates to core. This enables testing without mocking and reuse across adapters.
- **Plugin does NOT auto-start daemon**. User must start manually via `channel-mux daemon start`.
- **One bot token per daemon**. Multi-bot not supported in v1.
- **Unix socket only**. Windows native not supported (WSL works).
- **Access mutations only via terminal skill or CLI**, never from channel messages (prompt injection defense).
- **workspace:* protocol**: pnpm resolves to actual versions at publish time. CI uses `pnpm publish` (not npm) to ensure resolution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HealGaren)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HealGaren)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
