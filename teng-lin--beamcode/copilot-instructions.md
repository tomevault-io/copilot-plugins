## beamcode

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
pnpm build           # full build (lib + web)
pnpm build:lib       # TypeScript library only
pnpm typecheck       # tsc --noEmit
pnpm check:fix       # biome check --write src/

# Unit + integration tests (no credentials needed)
pnpm test                           # all unit + integration tests
pnpm exec vitest run -t "test name" # single test by name

# E2E smoke — binary in PATH, no API key (every PR)
pnpm test:e2e:smoke                 # all adapters

# E2E full — binary + API key or CLI OAuth, sends real prompts (nightly)
pnpm test:e2e:full                  # all adapters
pnpm test:e2e:<adapter>             # e.g. :claude :gemini :codex :opencode :agent-sdk

# Single e2e test with full tracing
BEAMCODE_TRACE=1 BEAMCODE_TRACE_LEVEL=full BEAMCODE_TRACE_ALLOW_SENSITIVE=1 \
  E2E_PROFILE=real-full USE_REAL_CLI=true \
  pnpm exec vitest run src/e2e/session-coordinator-gemini.e2e.test.ts \
  --config vitest.e2e.real.config.ts -t "test name" 2>trace.ndjson
pnpm trace:inspect   # analyze trace.ndjson
```

## Worktree Workflow

Always work in an isolated worktree, never directly on `main`.

```bash
# Create
git worktree add .worktrees/<name> -b <type>/<branch>
# e.g. git worktree add .worktrees/fix-gemini -b fix/gemini-e2e

---
> Source: [teng-lin/beamcode](https://github.com/teng-lin/beamcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
