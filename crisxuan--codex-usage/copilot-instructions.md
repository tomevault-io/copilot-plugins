## codex-usage

> macOS, Linux, or WSL:

# AI Agent Quick Start

## First Command

macOS, Linux, or WSL:

```bash
./run.sh
```

Windows PowerShell:

```powershell
.\run.ps1
```

## Useful Commands

macOS, Linux, or WSL:

```bash
./scripts/bootstrap.sh
./scripts/demo.sh
./scripts/check.sh
.venv/bin/codex-usage codex report --today
.venv/bin/codex-usage codex report --today --lang zh
.venv/bin/codex-usage codex chart --today --lang zh --output usage.html
.venv/bin/codex-usage --help
```

Windows PowerShell:

```powershell
.\scripts\bootstrap.ps1
.\scripts\demo.ps1
.\scripts\check.ps1
.\.venv\Scripts\codex-usage.exe codex report --today
.\.venv\Scripts\codex-usage.exe codex report --today --lang zh
.\.venv\Scripts\codex-usage.exe codex chart --today --lang zh --output usage.html
.\.venv\Scripts\codex-usage.exe --help
```

## Language

When reporting usage for a user, match the user's prompt language:

- Chinese prompt: pass `--lang zh`.
- English prompt: pass `--lang en`.
- Explicit language request: follow the requested language.
- No prompt context: use `--lang auto`.

## Real User Data

Real usage data lives in `.codex-usage/`. It is ignored by git. Do not delete or overwrite it unless the user explicitly asks.

Create a fresh local usage database:

```bash
.venv/bin/codex-usage init
```

```powershell
.\.venv\Scripts\codex-usage.exe init
```

## What This Tool Can and Cannot Do

Reads Codex local `token_count` logs.

Estimates imported transcript usage from locally visible text.

Does not read hidden prompts or subscription billing internals.

---
> Source: [crisxuan/codex-usage](https://github.com/crisxuan/codex-usage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
