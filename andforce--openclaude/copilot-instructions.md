## openclaude

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

This is a reconstructed source of Codex CLI (v2.1.88), restored from the `@anthropic-ai/Codex` npm package source map. The codebase is a TypeScript/React terminal application built with Bun.

## Build System

**Build Tool:** Bun v1.3.11+ (required for `bun:bundle` feature API)

**Package Manager:** pnpm

### Common Commands

```bash
# Build the project
bun run build.ts

# Run the CLI
bun dist/cli.js --version

# Development mode (run without building)
bun src/entrypoints/cli.tsx

# Install dependencies (optional, node_modules included)
pnpm install --registry https://registry.npmjs.org
```

### Build Configuration

The build is configured in `build.ts`:
- Entry point: `src/entrypoints/cli.tsx`
- Output: `dist/cli.js` (ES module, 22MB)
- Target: Node.js
- Source maps: linked

**Feature Flags:** 90+ compile-time feature flags defined in `build.ts`. Key flags include:
- `BRIDGE_MODE: false` - IDE bridge (disabled in production)
- `COORDINATOR_MODE: false` - Multi-agent coordination
- `BUILTIN_EXPLORE_PLAN_AGENTS: true` - Built-in exploration/planning agents
- `TOKEN_BUDGET: true` - Token budget display
- `MCP_SKILLS: true` - MCP skill support

**MACRO Constants:** Build-time constants injected via `define`:
- `MACRO.VERSION` - Version string ("2.1.88")
- `MACRO.BUILD_TIME` - ISO build timestamp
- `MACRO.ISSUES_EXPLAINER` - Support URL

## Architecture

### Entry Points

- **`src/entrypoints/cli.tsx`** - Main CLI entry point. Handles argument parsing, initialization, and launches the REPL or executes commands directly.
- **`src/main.tsx`** - Core REPL logic and command processing.

### Key Directory Structure

```
src/
├── entrypoints/     # Application entry points (cli.tsx, etc.)
├── commands/        # Slash commands (~103 files)
├── components/      # Terminal UI React components (~389 files)
│   ├── design-system/  # UI primitives
│   ├── messages/       # Message rendering
│   └── permissions/    # Permission dialogs
├── tools/           # Tool implementations (~184 tools)
│   ├── AgentTool/      # Subagent spawning
│   ├── BashTool/       # Shell execution
│   ├── File*Tool/      # File operations
│   └── ...
├── services/        # Core business logic (~130 files)
│   ├── mcp/            # MCP (Model Context Protocol)
│   ├── api/            # API clients
│   └── analytics/      # Telemetry/GrowthBook
├── utils/           # Utility functions (~564 files)
├── hooks/           # Lifecycle hooks (~104 files)
├── ink/             # Custom terminal rendering engine (~96 files)
├── types/           # Shared TypeScript types
└── vendor/          # Internal vendor code
```

### Core Concepts

**Tool System** (`src/Tool.ts`):
- Tools are the primary interface for LLM interactions
- Each tool implements: `call()`, `description()`, `inputSchema`, `checkPermissions()`, render methods
- Tools can be deferred (loaded on-demand via ToolSearch)
- Tools declare: `isReadOnly()`, `isDestructive()`, `isConcurrencySafe()`

**Task System** (`src/Task.ts`):
- Task types: `local_bash`, `local_agent`, `remote_agent`, `in_process_teammate`, `local_workflow`, `dream`
- Task IDs are prefixed: `b` (bash), `a` (agent), `r` (remote), `t` (teammate), etc.
- Tasks have lifecycle: `pending` → `running` → `completed|failed|killed`

**Permission System** (`src/types/permissions.ts`):
- Modes: `default`, `auto`, `bypass`
- Rules: `alwaysAllowRules`, `alwaysDenyRules`, `alwaysAskRules`
- Tools implement `checkPermissions()` for custom logic

**State Management** (`src/state/AppState.ts`):
- Centralized React-like state using immutable updates
- `setAppState()` for updates, `getAppState()` for reads
- Persisted to disk for session recovery

### Technology Stack

- **Runtime:** Bun (bundling + execution)
- **UI Framework:** React + Ink (terminal React renderer)
- **Type System:** TypeScript 5.x with Zod for runtime validation
- **API Client:** @anthropic-ai/sdk
- **MCP:** @modelcontextprotocol/sdk

### Private Package Stubs

The following internal Anthropic packages are stubbed (not on public npm):
- `@ant/Codex-for-chrome-mcp` - Chrome extension MCP
- `@anthropic-ai/sandbox-runtime` - Sandbox runtime
- `@anthropic-ai/bedrock-sdk`, `@anthropic-ai/foundry-sdk`, `@anthropic-ai/vertex-sdk` - Cloud providers
- `@anthropic-ai/mcpb` - MCP bundle processor
- `color-diff-napi` - Syntax highlighting native module
- `modifiers-napi` - macOS key modifier detection

### Commander.js Patch

The build requires a patched `node_modules/commander/lib/option.js` to support multi-character short options (e.g., `-d2e`). The regex `/^-[^-]$/` is changed to `/^-[^-]+$/`.

## Development Notes

- **Bun-specific code:** Heavy use of `bun:bundle` `feature()` API for compile-time dead code elimination
- **File watching:** Uses `chokidar` for file system watching
- **Terminal handling:** Custom Ink-based rendering engine in `src/ink/`
- **Vim mode:** Custom implementation in `src/vim/`

## Testing

No test suite is included in the reconstructed source. Testing would require:
1. Setting up a test framework (Bun's built-in test runner)
2. Creating mock implementations of Anthropic API clients
3. Stubbing native modules

---
> Source: [andforce/Openclaude](https://github.com/andforce/Openclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
