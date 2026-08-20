## ai-report-to-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

AI Report to Me (`aireport`) is a CLI tool that reads coding agent session logs (Claude Code, Gemini CLI, OpenCode, Codex) from the local filesystem, parses them into a normalized format, analyzes patterns (friction, habits, coder type), and generates daily reports and "Wrapped" summaries (like Spotify Wrapped but for coding sessions).

## Commands

- `pnpm dev -- <subcommand>` — run CLI in dev mode via tsx
- `pnpm build` — build with tsup (outputs to `dist/cli/`)
- `pnpm test` — run all tests with vitest
- `pnpm test -- src/__tests__/parser.test.ts` — run a single test file
- `pnpm test:watch` — run tests in watch mode
- `pnpm typecheck` — type-check without emitting

## Architecture

The data flows in a pipeline: **Adapters → SessionReader → Aggregator → Analyzers → Renderer → Output**

### Adapters (`src/adapters/`)
Each supported CLI tool has an adapter implementing `CLIAdapter` interface. Adapters handle detection, session listing, JSONL parsing, and hook installation. Currently only `claude-code` is implemented. New CLI adapters register in `AdapterRegistry` (`registry.ts`).

### Core (`src/core/`)
- **SessionReader** — collects `NormalizedSession[]` from all enabled adapters, groups by day
- **Aggregator** — computes `DailyAggregation` or `WrappedAggregation` from sessions (project breakdown, tool call distribution, hourly patterns, token usage)
- **ReportScheduler** — determines which days need report (re)generation by comparing session IDs against existing report metadata
- **Analyzers** (`core/analyzer/`) — friction detection (retry loops, error loops, direction switches), habit patterns, vibe coder type classification, improvement suggestions
- **Prompts** (`core/prompts/`) — LLM prompt templates for daily and wrapped reports

### Shared (`src/shared/`)
- **types.ts** — all core data types (`NormalizedSession`, `NormalizedMessage`, `ToolCall`, `DailyAggregation`, `WrappedAggregation`, etc.)
- **config.ts** — Zod-validated config at `~/.ai-report/config.json` (language, privacy mode, backfill limit, webhook URLs)
- **storage.ts** — JSONL/Markdown read/write helpers, report metadata embedding/parsing
- **constants.ts** — paths (`~/.ai-report/`, `~/.claude/`), defaults

### Renderer (`src/renderer/`)
Markdown renderers for daily and wrapped reports.

### Output (`src/output/`)
Output targets implementing `OutputTarget` interface (currently local file).

### CLI (`src/cli/`)
Commander-based CLI with subcommands: `status`, `config`, `daily`, `wrapped`, `install`, `uninstall`, `log-session`.

## Publishing

1. Bump version in `package.json`
2. `pnpm build`
3. `npm publish --registry=https://registry.npmjs.org --access public --//registry.npmjs.org/:_authToken=$NPM_PUBLISH_TOKEN`

Note: Default registry in `~/.npmrc` points to npmmirror, so `--registry` is required. The token is a granular access token with 2FA bypass (no OTP needed), stored in auto memory, not in the repo.

## Key Conventions

- ESM-only (`"type": "module"` in package.json), all imports use `.js` extensions
- TypeScript strict mode, target ES2022, Node 20+
- Path alias `@/*` maps to `./src/*` (tsconfig only, tsup bundles everything)
- Tests live in `src/__tests__/`, use vitest with globals enabled
- All data stored under `~/.ai-report/` (config, reports, session log)
- Session data is read from CLI-specific directories (e.g., `~/.claude/projects/`)
- Reports embed JSON metadata in HTML comments (`<!-- ai-report-meta ... -->`) for incremental regeneration
- Specs/PRD docs are in `specs/` (Chinese language)

---
> Source: [FakeHank/ai-report-to-me](https://github.com/FakeHank/ai-report-to-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
