## copilot-local

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build      # Compile TypeScript to dist/ (adds shebang + sets executable bit)
npm run lint       # Biome static analysis
npm run format     # Biome auto-format (run before lint to fix formatting violations)
npm test           # Vitest test suite
```

Run a single test by name:
```bash
npx vitest run --reporter=verbose -t "test name substring"
```

## Architecture

Single-file CLI (`src/index.ts`) with three commands, built on the `cac` argument parser:

- **Default command `<model>`** — loads a named model profile from `~/.config/copilot-local/settings.json`, injects its environment variables, then spawns the `copilot` subprocess. Defaults to interactive mode (`-i ""`) when no extra args are given.
- **`generate-config`** — fetches `/v1/models` from an OpenAI-compatible endpoint (Ollama, llama.cpp, LM Studio, etc.) and prints a JSON config template to stdout.
- **`completions`** — emits zsh/bash completion scripts; `--list` prints available model names for use inside completion scripts.

The config file schema is a plain JSON object keyed by model name, where each value is a flat map of environment variable names to string values. The CLI validates that all keys are legal env var names before launching the subprocess.

`dist/index.js` is the published binary entry point (`bin: copilot-local`). The `postbuild` script prepends the Node.js shebang after `tsc` runs.

Tests (`src/index.test.ts`) use Vitest and spin up a real HTTP server to exercise `generate-config`; everything else uses direct module imports with no mocking of the file system.

## Tech stack

- TypeScript → ES2024 / NodeNext modules, strict mode
- Biome for lint + format (2-space indent, double quotes)
- Vitest for tests
- Node.js 22+ required at runtime

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blauerberg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
