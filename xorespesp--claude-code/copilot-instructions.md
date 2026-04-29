## claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Reconstructed TypeScript source tree of the Claude Code CLI, recovered from source maps of the public npm release. Some native modules are replaced with compatible shims in `shims/`. This is **not** the upstream Anthropic development repo.

## Development Commands

```bash
bun install        # Install dependencies (includes local shim packages)
bun run dev        # Start CLI interactively (alias: bun run start)
bun run version    # Verify the CLI boots and prints version
```

**Requirements:** Bun >= 1.3.5, Node.js >= 24

There is no lint, test, or build script. Validation is manual: boot the CLI and exercise the changed path.

## Boot Sequence

`src/dev-entry.ts` is the entry point. It scans for missing relative imports across `src/` and `vendor/`. If any are missing, it prints a diagnostic and exits. When all imports resolve, it forwards to `src/entrypoints/cli.tsx` → `src/main.tsx` which runs full initialization (auth, GrowthBook, MCP, settings, Commander.js) and launches the React/Ink REPL.

## Architecture

- **`src/main.tsx`** — Full CLI initialization: side-effect imports for startup profiling, MDM reads, and keychain prefetch must stay at the top and in order.
- **`src/query.ts` / `src/QueryEngine.ts`** — Streaming API call loop, tool execution, auto-compaction, session lifecycle, and persistence.
- **`src/commands.ts`** — Slash command registration (`/commit`, `/review`, `/config`, `/compact`, `/memory`, `/plan`, etc.).
- **`src/tools/`** — 53 tool implementations, each in its own directory (e.g., `BashTool/`, `FileEditTool/`, `AgentTool/`).
- **`src/commands/`** — 87 slash command implementations, each in its own directory with an `index.ts` entry.
- **`src/services/`** — Backend services: API client (`api/`), MCP protocol (`mcp/`), session compaction (`compact/`), analytics/feature flags (`analytics/`).
- **`src/components/`** — Terminal UI components built with React + Ink (messages, inputs, diffs, permission dialogs, status bar).
- **`src/hooks/`** — Custom React hooks for tools, voice, IDE integration, vim mode, sessions, tasks.
- **`src/ink/`** — Custom Ink fork handling layout, focus, ANSI rendering, virtual scrolling, and click detection.
- **`src/vim/`** — Full vim keybinding engine (motions, operators, text objects).
- **`src/coordinator/`** — Multi-agent coordination, conditionally loaded via `bun:bundle` feature flag `COORDINATOR_MODE`.
- **`src/assistant/`** — KAIROS assistant mode, conditionally loaded via feature flag `KAIROS`.
- **`src/bridge/`** — Remote bridge control for web/IDE connections.
- **`shims/`** — Compatibility replacements for private native modules (`color-diff-napi`, `modifiers-napi`, `url-handler-napi`, and Anthropic-internal MCP packages).
- **`vendor/`** — Native binding source code (`audio-capture-src`, `image-processor-src`, `modifiers-napi-src`, `url-handler-src`).

## Code Style

- TypeScript with ESM imports and `react-jsx`. `strict` mode is off in tsconfig.
- Most files omit semicolons and use single quotes — match surrounding file style exactly.
- `camelCase` for variables/functions, `PascalCase` for React components and classes, `kebab-case` for command directories.
- Import order: when comments warn against reordering, keep imports stable. `main.tsx` side-effect imports at the top are order-sensitive.
- Path alias: `src/*` maps to `./src/*` via tsconfig paths.
- Prefer small, focused modules over broad utility dumps.

## Testing & Validation

There is no automated test suite. For any change:

1. Boot the CLI with `bun run dev`.
2. Smoke-test version output with `bun run version`.
3. Exercise the specific command, service, or UI path you changed.

When adding tests, place them close to the feature they cover and name them after the module or behavior under test.

## Commit Style

Use short, imperative commit subjects (e.g., `Fix MCP config normalization`). In PRs, explain user-visible impact and list validation steps.

## Key Constraints

- `dev-entry.ts` gates startup on zero missing imports. Any new module must be resolvable or the CLI won't boot.
- Conditional imports via `bun:bundle` feature flags (`COORDINATOR_MODE`, `KAIROS`) use `require()` to avoid circular dependencies — preserve this pattern.
- Changes to shims should maintain API compatibility with the original native modules they replace.
- This is a reconstructed source tree — prefer minimal, auditable changes. Document any workaround added because a module was restored with fallbacks or shim behavior.

---
> Source: [xorespesp/claude-code](https://github.com/xorespesp/claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
