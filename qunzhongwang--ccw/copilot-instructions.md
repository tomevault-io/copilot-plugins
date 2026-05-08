## ccw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
source ~/.nvm/nvm.sh    # nvm must be sourced first on this system
npm run build           # tsc + shebang injection for dist/index.js and dist/health.js
npm run dev             # tsc --watch
```

No test runner or linter is configured yet.

## What This Project Does

`ccw` is a drop-in replacement for `claude` that checks the user's IP geolocation before launch. If the user is in the US, it launches Claude Code normally. If not, it injects API credentials (OpenRouter, Poe, etc.) from `~/.claude/.env` so Claude Code uses an alternative provider. A proxy-first strategy is supported: if a configured proxy makes the IP appear US, claude launches normally through that proxy without needing API keys.

There are two CLI entry points:
- **`ccw`** (dist/index.js) — the wrapper that replaces `claude`
- **`claude-health`** (dist/health.js) — interactive config wizard

## Architecture

The codebase is six modules in `src/`, each with a single responsibility:

- **index.ts** — CLI entry point for `ccw`. Parses own flags (`--skip-check`, `--dry-run`, `--provider`), orchestrates the flow: load config → proxy-first IP check → direct IP check → resolve provider → launch.
- **health.ts** — CLI entry point for `claude-health`. Interactive wizard that walks users through proxy setup, provider configuration, and saves to `~/.claude/.env`.
- **config.ts** — Reads/writes `~/.claude/.env` using a custom INI-style parser. Returns `AppConfig { global: GlobalConfig, providers: Record<string, Provider> }`. Lines before any `[section]` are global; each `[section]` is a named provider.
- **ip-check.ts** — Single function `checkIP(proxy?)` that hits `ipapi.co/json/` with a 5s timeout. Returns `{ country, countryCode, ip }` or `null` on failure.
- **launcher.ts** — Finds `claude` via `which`, spawns it with `stdio: "inherit"` and optional extra env vars, forwards signals and exit codes.
- **prompt.ts** — Readline-based interactive prompts: `pickProvider()`, `askYesNo()`, `askInput()`, `askChoice()`. All check `process.stdin.isTTY` before prompting.

## Key Design Decisions

- **Safe default on IP check failure**: any error (timeout, no internet, bad response) falls through to API key mode rather than blocking the user.
- **Environment injection**: provider credentials are mapped to `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`, `ANTHROPIC_MODEL` — the env vars Claude Code reads natively.
- **Zero runtime deps beyond proxy agents**: config parsing, prompts, and logging are all hand-rolled to keep the package minimal. Only `https-proxy-agent` and `http-proxy-agent` are runtime dependencies.
- **All output goes to stderr**: logger functions use `console.error` so they don't interfere with claude's stdout.

---
> Source: [qunzhongwang/ccw](https://github.com/qunzhongwang/ccw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
