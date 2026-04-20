## agent-foreman

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Build
pnpm run build              # Full build (embed assets + TypeScript compile)
pnpm run build:embed        # Generate embedded assets only
pnpm run build:bin          # Build standalone binary

# Testing (always use CI=true)
CI=true pnpm test                                  # Run all tests
CI=true pnpm run test:watch                        # Watch mode
CI=true npx vitest tests/path/to/file.test.ts     # Single test file

# Development
pnpm run dev                # Run CLI directly via tsx (no build needed)
pnpm run start              # Run compiled CLI from dist/

# Type checking
npx tsc --noEmit            # Check types without emitting
```

## Architecture Overview

CLI tool built with TypeScript + yargs for task-driven AI agent development.

### Entry Point Flow

```text
src/index.ts → src/commands/index.ts (yargs) → src/commands/cli-commands.ts (registration)
```

### Core Modules

| Module | Purpose |
|--------|---------|
| `src/commands/` | CLI commands: init, next, done, check, status, impact, fail, tdd, install, uninstall |
| `src/features/` | Task CRUD, selection, mutations, statistics |
| `src/storage/` | Markdown parsing/serialization for `ai/tasks/*.md` |
| `src/strategies/` | Universal Verification Strategy (UVS) executors |
| `src/verifier/` | Verification orchestration |
| `src/agents/` | AI agent detection and orchestration |
| `src/scanner/` | Project scanning and survey generation |
| `src/capabilities/` | AI capability discovery and caching |
| `src/tdd-guidance/` | TDD workflow guidance generation |

### Verification Strategy Pattern

Registry pattern in `src/strategies/index.ts`:

- Strategies implement `StrategyExecutor` interface from `src/strategy-executor.ts`
- Auto-register via side-effect imports when module loads
- Types: `test`, `e2e`, `script`, `http`, `file`, `command`, `manual`, `ai`, `composite`
- Each strategy has its own directory (e.g., `src/strategies/ai-strategy/`) with executor, types, and output modules

### Storage Format

Tasks stored as Markdown with YAML frontmatter:

- Index: `ai/tasks/index.json` (lightweight lookup)
- Content: `ai/tasks/{module}/{id}.md` (full definition)

### Key Types

`src/types/index.ts` exports:

- `Feature`/`Task` - Task definition
- `FeatureList` - Collection with metadata
- `FeatureIndex` - Lightweight index

### Schema Validation

JSON schemas in `src/schemas/` with AJV validation:

- `feature-list.ts` - Legacy JSON format validation
- `feature-frontmatter.ts` - Markdown frontmatter validation
- `feature-index.ts` - Index file validation

## Testing Notes

- Tests use vitest with `forks` pool for process isolation
- 30s default timeout, 5s teardown timeout
- Global setup in `tests/setup.ts` handles cleanup hooks
- Coverage excludes `src/index.ts` (entry point)

## Release Process (Human Only)

```bash
npm version patch && git push origin main --tags
```

Version auto-syncs across `package.json`, `marketplace.json`, and `plugin.json`.

## Workflow Rules

Agent workflow rules are in `.claude/rules/` (symlinked to `src/rules/templates/`).

## Project Goal

Long Task Harness for AI agents - task/feature-driven development with external memory

---
> Source: [mylukin/agent-foreman](https://github.com/mylukin/agent-foreman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
