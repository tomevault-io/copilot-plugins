## contextanalyzer

> CLI tool that connects to OpenCode, reads session history, and renders terminal histograms showing where context tokens and wall-clock time are spent. Helps debug which tool calls consume the most context and which are slowest.

# contextanalyzer

CLI tool that connects to OpenCode, reads session history, and renders terminal histograms showing where context tokens and wall-clock time are spent. Helps debug which tool calls consume the most context and which are slowest.

## Before starting

Load these skills first:

- **goke**: this project uses `goke` for CLI argument parsing with Zod schemas
- **errore**: errors are returned as values (`Error | T` unions), not thrown. Check `instanceof Error` before using results
- **changesets**: add a `.changeset/*.md` file for every user-facing change or fix

## Architecture

- `src/cli.ts`: entry point, goke CLI definition, orchestrates connection → session selection → analysis → render
- `src/opencode-client.ts`: spawns `opencode serve`, connects via `@opencode-ai/sdk`, provides session listing and message fetching
- `src/analyze.ts`: core analysis engine, processes `MessageWithParts[]` from the SDK into `AnalysisResult`
- `src/render.ts`: ASCII histogram and summary rendering for terminal output
- `src/bash-command.ts`: extracts first command name from bash strings for grouping

Token computation uses real data from the OpenCode SDK (`AssistantMessage.tokens`), not character-based estimation. Total prompt tokens = `input + cache.read + cache.write` (OpenCode reports `input` as uncached tokens only). Cost is summed across all assistant messages.

## Stack

- TypeScript ESM, built with `tsc`
- **pnpm** for package management
- `goke` for CLI, `zod` for option schemas, `@clack/prompts` for interactive session picker
- `@opencode-ai/sdk` for session data via HTTP API
- `just-bash` for bash command parsing

## Build and run

```bash
pnpm build        # tsc + chmod
pnpm dev          # tsx src/cli.ts
```

---
> Source: [remorses/contextanalyzer](https://github.com/remorses/contextanalyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
