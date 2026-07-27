## andro-cfw

> Repository guide for Google Antigravity (AGY) & Gemini coding agents.

# GEMINI.md — andro-cfw Context & Conventions

Repository guide for Google Antigravity (AGY) & Gemini coding agents.

## Project Summary
`andro-cfw`: Python library & CLI tool deploying Cloudflare Workers as unfiltered reverse proxies and 24/7 serverless webhook engines for Telegram bots in restricted regions (e.g. Iran). Zero VPN required.

## Toolchain & Stack
- **Language**: Python 3.9+ (Core CLI/library), TypeScript (Cloudflare Worker template)
- **Package Manager**: `uv` for Python (`uv pip install`, `uv run`, `uv build`), `bun` / `npx wrangler` for JS/TS worker toolchain
- **Testing**: `uv run --with pytest pytest` (76 unit tests in `tests/`)
- **Code Search**: `rg` (ripgrep)
- **Secrets**: Never commit raw tokens or Fernet session keys

## Key Files & Layout
- `andro_cfw/patcher.py`: `andro_cfw.patch()` 1-line auto-patcher for `telebot`, `pyrogram`, `hydrogram`, `aiogram`, `telegram`
- `andro_cfw/session.py`: Encrypted `cfw.session` storage & health diagnostics (`check_health()`)
- `andro_cfw/loadbalancer.py`: Local HTTP load balancer for multi-account Cloudflare pool failover
- `andro_cfw/templates/worker.ts`: Dual-mode Cloudflare Worker template (reverse proxy + 24/7 serverless webhook)
- `andro_cfw/cli.py`: CLI entry points (`init`, `serverless`, `add-account`, `check`, `status`, `snippet`, `remove`, `setup-path`)
- `plans/`: Monotonic implementation plan markdown files

## Verification Commands
```bash
uv run --with pytest pytest
```
All code edits must maintain 100% test suite pass rate.

---
> Source: [Andromeda-Collective/andro-cfw](https://github.com/Andromeda-Collective/andro-cfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
