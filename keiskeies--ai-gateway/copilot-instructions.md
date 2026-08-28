## ai-gateway

> AI Gateway: a Rust + React + Tauri desktop/server app that load-balances requests across multiple AI platforms (OpenAI, Anthropic, DeepSeek, etc.). Exposes OpenAI-compatible `/v1/chat/completions` and Anthropic-compatible `/v1/messages` endpoints.

# AGENTS.md

## Project overview

AI Gateway: a Rust + React + Tauri desktop/server app that load-balances requests across multiple AI platforms (OpenAI, Anthropic, DeepSeek, etc.). Exposes OpenAI-compatible `/v1/chat/completions` and Anthropic-compatible `/v1/messages` endpoints.

## Architecture

- **Two Rust crates**: root crate (`src/`) is both a library (`ai_gateway`) and a standalone binary (`ai-gateway`); `src-tauri/` is the Tauri desktop wrapper that depends on the root crate via `path = ".."`.
- **Frontend**: React + TypeScript + Ant Design + Vite in `frontend/`. Builds to `../static/` (NOT `frontend/dist/`), which the Rust binary serves as static files.
- **Database**: SQLite via rusqlite + r2d2. DB path resolves relative to app dir (`config.toml` location).
- **Config**: `config.toml` in app directory. Loaded via `AppConfig::load_or_default()` which searches upward from exe dir for `config.toml` (handles macOS .app bundles and Windows install paths).

## Build & run commands

```bash
# Standalone server mode (serves on port 1994)
cargo run

# Frontend dev server (port 5173, proxies /api /v1 /health to localhost:1994)
cd frontend && npm install && npm run dev

# Build frontend (outputs to ../static/)
cd frontend && npm run build

# Tauri desktop app
cargo install tauri-cli
cargo tauri dev

# Build release
cargo build --release
```

## Key ports

- **1994**: Rust backend (API + static file server). Configurable in `config.toml`.
- **5173**: Vite dev server (frontend only, proxies API calls to 1994).

## No test infrastructure

There are no unit tests, integration tests, or test commands configured. CI (`release.yml`) only builds and creates GitHub releases — no test step.

## Code conventions

- **Chinese comments** are used throughout the Rust backend. Keep comments in Chinese when editing existing files.
- **Rust style**: actix-web handlers, `AppError` enum with `From` impls, `AppResult<T>` type alias. Modules organized by domain: `api/`, `db/`, `proxy/`, `lb/`, `protocol/`.
- **API routes** registered in `src/api/mod.rs` under `/api` scope. Proxy endpoints at root level (`/v1/chat/completions`, `/v1/messages`, `/v1/models`).
- **Frontend**: React functional components, Ant Design components, i18n via `src/i18n.ts` (zh/en). Pages in `src/pages/`, API calls in `src/api.ts`.
- **No linting/formatting tools** configured for Rust or TypeScript (no clippy, rustfmt, eslint, prettier).
- **Dev profile**: `opt-level = 2` for all dependencies (faster dev builds at cost of compile time).

## Gotchas

- Frontend builds to `../static/` not `frontend/dist/` — the `emptyOutDir: true` in `vite.config.ts` means `static/` gets wiped on each frontend build.
- The Rust binary finds static files and config by searching upward from the executable directory for `config.toml`. This path resolution is critical for Tauri bundles on macOS (Contents/Resources/_up_/).
- Config resolution searches upward from exe dir. On Windows it may find `C:\Users\<user>\AppData\Local\AI Gateway\config.toml` before the project-local one — that becomes the active config and DB path.
- The `config.toml` `encrypt_key` and `admin_token` are security-sensitive. Never commit real values.
- No `data/` directory or `*.db` files should be committed (gitignored).

---
> Source: [keiskeies/ai-gateway](https://github.com/keiskeies/ai-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
