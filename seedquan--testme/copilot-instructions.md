## testme

> testme is an AI-powered product tester CLI. It reads a GitHub repo's README, spins up a Docker sandbox, runs Claude Code inside to install and test the product as a real user, then files GitHub issues with findings.

# CLAUDE.md — testme contributor guide

## What is this project?

testme is an AI-powered product tester CLI. It reads a GitHub repo's README, spins up a Docker sandbox, runs Claude Code inside to install and test the product as a real user, then files GitHub issues with findings.

## Dev workflow

```bash
npm install          # install deps
npm run test         # run 108 tests (vitest)
npm run build        # build with tsup + chmod
npm run dev -- <url> # run from source
```

## Project structure

```
src/
  cli.ts                  # CLI entry, subcommands (init, cleanup, reports, diff)
  config.ts               # Config types, .testmerc.json loader
  version.ts              # Version from package.json
  github/                 # GitHub API: fetch context, create issues
  docker/                 # Docker container lifecycle
  orchestrator/           # Main flow: plan → execute → parse → report
  prompt/                 # System prompts for Claude Code
  output/                 # Terminal output, JSON/markdown reports
  diff/                   # Report comparison for regression detection
  utils/                  # Retry, error formatting
```

## Key conventions

- TypeScript strict mode, ESM (`"type": "module"`)
- Tests colocated: `foo.ts` → `foo.test.ts`
- No `any` types — use `unknown` and narrow
- Build produces single `dist/index.js` via tsup
- Config cascade: explicit CLI flag > .testmerc.json > env var > default
- Exit codes: 0=clean, 1=findings, 2=error

## Testing

All tests use vitest. Run `npm run test` before committing. Tests cover:
- URL parsing, config loading, findings parsing, report formatting
- String similarity (Levenshtein), JSON extraction
- Error formatting, retry logic, report diffing
- CLI config cascade integration tests

## Docker

The Dockerfile (`Dockerfile.testme`) builds the sandbox image with Node, Python, Chromium, Playwright, and Claude Code. Global npm packages install as root before the `USER tester` switch.

## Adding a CLI subcommand

Add it in `src/cli.ts` using `program.command("name")`. Follow the pattern of existing commands (init, cleanup, reports, diff).

---
> Source: [seedquan/testme](https://github.com/seedquan/testme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
