## freecc

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project

Free CC — Unleash your full power with Claude Code. Multi-provider support, self-hosted remote web access, OpenAI integration, and all experimental features unlocked.

## Common commands

```bash
# Install dependencies
bun install

# Standard build (./freecc)
bun run build

# Dev build (./freecc-dev)
bun run build:dev

# Dev build with all experimental features
bun run build:dev:full

# Compiled build (./dist/freecc)
bun run compile

# Run from source without compiling
bun run dev
```

Run the built binary with `./freecc` or `./freecc-dev`. Set `ANTHROPIC_API_KEY` in the environment or use OAuth via `./freecc /login`.

## High-level architecture

- **Entry point/UI loop**: src/entrypoints/cli.tsx bootstraps the CLI, with the main interactive UI in src/screens/REPL.tsx (Ink/React).
- **Command/tool registries**: src/commands.ts registers slash commands; src/tools.ts registers tool implementations. Implementations live in src/commands/ and src/tools/.
- **LLM query pipeline**: src/QueryEngine.ts coordinates message flow, tool use, and model invocation.
- **Core subsystems**:
  - src/services/: API clients, OAuth/MCP integration, analytics stubs
  - src/state/: app state store
  - src/hooks/: React hooks used by UI/flows
  - src/components/: terminal UI components (Ink)
  - src/skills/: skill system
  - src/plugins/: plugin system
  - src/bridge/: IDE bridge
  - src/voice/: voice input
  - src/tasks/: background task management

## Custom features (Free CC additions)

### /provider command
- `src/commands/provider/` — switch API providers (Anthropic, Bedrock, Vertex, Foundry, OpenAI)
- OpenAI supports both Codex OAuth and API key + custom base URL
- Config stored in GlobalConfig (`src/utils/config.ts`): `openaiApiKey`, `openaiBaseUrl`
- Codex fetch adapter: `src/services/api/codex-fetch-adapter.ts` (has `createOpenAIApiFetch`)

### /remote-connect command
- `src/commands/remote-connect/` — connect to a self-hosted relay server
- `src/remote-server/client.ts` — WebSocket client transport
- `src/remote-server/relay.ts` — global singleton that polls messagesRef and forwards to web
- REPL integration: 2 lines in `src/screens/REPL.tsx` (search for "Remote relay")
- Supports client pairing (no pre-shared key needed)

### Remote relay plugin
- `src/plugins/bundled/remote-relay/` — builtin plugin for discoverability/toggle
- Registered in `src/plugins/bundled/index.ts`

### Server (separate repo)
- https://github.com/chat812/freecc-server
- Node.js WebSocket relay with admin dashboard, pairing, session persistence

## Build system

- scripts/build.ts is the build script and feature-flag bundler
- Binary output: `./freecc` (production), `./freecc-dev` (dev)
- Feature flags set via build arguments (e.g., `--feature=ULTRAPLAN`) or presets like `--feature-set=dev-full`

## Key patterns

- REPL.tsx is React Compiler output — hooks use `$` cache array. Adding raw `useEffect` works but avoid modifying compiled `useCallback` internals.
- Messages are wrapped: actual content is at `msg.message.content`, not `msg.content`.
- Message types: `user` (not `human`), `assistant`, `tool_result`, `system`, `progress`, `attachment`.
- Commands register in `src/commands.ts` via import + add to `COMMANDS()` array.
- Builtin plugins register in `src/plugins/bundled/index.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chat812) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
