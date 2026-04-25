## inserter

> Python + Rust tools that bulk-insert generated records into CrateDB for performance testing. Both use async I/O, gzip compression, and connection pooling.

# Project: CrateDB Record Generator (inserter)

## What this is
Python + Rust tools that bulk-insert generated records into CrateDB for performance testing. Both use async I/O, gzip compression, and connection pooling.

## Key files
- `crate_write/main.py` — Python async engine (aiohttp)
- `rust/src/main.rs` — Rust worker loop (tokio)
- `rust/src/client.rs` — HTTP client (reqwest, spawn_blocking for CPU work)
- `rust/config.toml` — Rust default config (auto-detected)
- `.env` / `rust/.env` — connection strings (gitignored)
- `BENCHMARKS.md` — performance comparison data

## Conventions
- Python: use `uv` for all Python tooling (venv, pip, run)
- Rust: config.toml values are defaults, CLI args override only when explicitly passed
- Commits: always include `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>`
- PRs: create branch, push, `gh pr create`, `gh pr merge --merge --delete-branch`
- Direct pushes to main are OK for small fixes

## When user says "update progress log"
Read `progress_log.md`, append current session's changes in the same terse changelog style. Keep only the last 5-6 sessions. Drop the oldest if needed. Keep total under ~50 lines of content.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WalBeh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
