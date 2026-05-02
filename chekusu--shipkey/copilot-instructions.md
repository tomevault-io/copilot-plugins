## shipkey

> CLI tool and core library for managing developer API keys securely.

# shipkey

CLI tool and core library for managing developer API keys securely.

**Repo:** `chekusu/shipkey`
**Role:** Core product code — CLI binary, `@shipkey/core` npm package, provider detection, secret backends.

## Repository Structure

- `src/` — CLI source code (commands: setup, push, pull, sync, list, scan)
- `packages/core/` — `@shipkey/core` shared library (backends, providers, types)
- `.github/workflows/release.yml` — Builds CLI binaries + publishes `@shipkey/core` to npm

## What belongs here

- CLI commands and logic
- `@shipkey/core`: secret backends (1Password, Bitwarden), provider detection/registry
- Release workflows for CLI binary and npm packages
- Tests

## What does NOT belong here

- Web landing page, marketing UI → goes to `chekusu/shipkey.dev`
- Setup wizard frontend components → goes to `chekusu/shipkey.dev`
- Web deployment workflows → goes to `chekusu/shipkey.dev`

---
> Source: [chekusu/shipkey](https://github.com/chekusu/shipkey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
