## gclm-code

> This file provides guidance to Gclm Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Gclm Code (claude.ai/code) when working with code in this repository.

## Common commands

```bash
# Install dependencies
bun install

# Standard build (dist/cli.js + dist/gclm)
bun run build

# Dev build (dist/cli.js + dist/gclm-dev)
bun run build:dev

# Dev build with all experimental features (dist/gclm-dev)
bun run build:dev:full

# Compiled build (./gclm)
bun run compile

# Run from source without compiling
bun run dev

# Typecheck
bun run typecheck

# Run all tests
bun test --preload ./tests/preload.ts tests/

# Run a single test file
bun test tests/orchestration/hello2cc.test.ts

# Run a single test by name
bun test --test-name-pattern "route guidance signature"

# Watch mode
bun run test:watch

# Smoke test the built binary
bun run smoke

# Verify build + smoke
bun run verify

# NPM package packing + smoke
bun run pack:npm
bun run smoke:npm

# Dev server (gclm-code-server)
bun run dev:gclm-code-server
```

Run the built binary with `./dist/gclm` or `./dist/gclm-dev`. Set `ANTHROPIC_API_KEY` in the environment or use OAuth via `./dist/gclm /login`.

## Release process

Release is driven entirely by GitHub Actions. To publish a new version:

1. Bump `version` in `package.json`
2. Commit and push to `main`
3. Tag with `git tag v<version>` and `git push origin v<version>`

Pushing a `v*` tag triggers `.github/workflows/release-npm.yml`, which builds, tests, runs smoke tests, publishes to npm, and sets the `stable` dist-tag. Use `workflow_dispatch` with `publish_to_npm=false` for dry runs.

## High-level architecture

### Entry point and UI loop
- `src/entrypoints/cli.tsx` bootstraps the CLI
- `src/screens/REPL.tsx` (Ink/React) is the main interactive UI
- `src/ink/` provides Ink-specific utilities and components

### Core request pipeline
- `src/QueryEngine.ts` coordinates message flow, tool use, and model invocation
- `src/query/` contains the lower-level query handling
- `src/cost-tracker.ts` tracks token usage and cost

### Command and tool registries
- `src/commands.ts` centralizes slash command registration (`/commit`, `/config`, etc.)
- `src/tools.ts` centralizes tool implementations
- Each command lives in `src/commands/<name>/` with an `index.ts` barrel
- Each tool lives in `src/tools/<ToolName>/` as a self-contained directory

### Key subsystems

| Directory | Responsibility |
|-----------|---------------|
| `src/bootstrap/` | Global state management, session lifecycle, hook registration |
| `src/services/` | API clients, OAuth, MCP integration |
| `src/state/` | AppState store (React state for the REPL) |
| `src/components/` | Reusable terminal UI components (Ink) |
| `src/utils/` | Shared utilities (settings, session storage/restore, auth, model config) |
| `src/types/` | Shared TypeScript type definitions |
| `src/hooks/` | React hooks used by UI/flows |
| `src/orchestration/` | Gateway capability orchestration (hello2cc) |
| `src/skills/` | Skill system for reusable workflows |
| `src/plugins/` | Plugin system for extensibility |
| `src/bridge/` | IDE bridge for editor integration |
| `src/voice/` | Voice input support |
| `src/tasks/` | Background task management |
| `src/gclm-code-server/` | Remote server implementation |
| `src/schemas/` | JSON schema definitions |
| `src/migrations/` | Data migration scripts |

### Build system

`scripts/build.mjs` is the unified build script and feature-flag bundler. It supports:
- `--dev` — development build with extra debugging
- `--compile` — compile to a standalone binary
- `--feature=X` — enable a specific feature flag
- `--feature-set=dev-full` — enable all experimental features

Feature flags control conditional imports (via `bun:bundle`), allowing the same codebase to produce different builds.

### Test infrastructure

Tests use Bun's native test runner (`bun:test`). Test files live under `tests/` and must be run with `--preload ./tests/preload.ts` to initialize the bootstrap state. Key test directories:
- `tests/orchestration/` — hello2cc orchestration tests
- `tests/tools.test.ts` — tool system tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gclm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
