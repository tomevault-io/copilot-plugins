## leader-stream

> This repo is a single Rust/Axum service with a WASM client.

# Repository Guidelines

## Project Structure & Module Organization
This repo is a single Rust/Axum service with a WASM client.
- `leader-stream/src/main.rs` hosts the Axum server, API handlers, and SSE.
- `leader-stream/src/lib.rs` + `leader-stream/src/wasm_app.rs` compile to the WASM client.
- `leader-stream/public/` contains static assets (`styles.css`, `og.svg`, generated `app.js` + `app_bg.wasm`).
- `k8s/` contains deployment manifests for the unified service.

## Build, Test, and Development Commands
- Dev server: `cargo run` inside `leader-stream/` (serves `http://localhost:3000`).
- Build WASM assets (manual):
  - `rustup target add wasm32-unknown-unknown`
  - `cargo build --release --target wasm32-unknown-unknown --lib` (from `leader-stream/`)
  - `wasm-bindgen --target no-modules --out-name app --out-dir ./public ./target/wasm32-unknown-unknown/release/leader_stream.wasm`
  - `printf '\nwasm_bindgen(\"/app_bg.wasm\");\n' >> ./public/app.js`
- Production build: `make build` (Docker image).

## Coding Style & Naming Conventions
- Rust uses `rustfmt` defaults.
- CSS uses 4-space indentation; keep class names in kebab-case.
- Prefer CSS variables in `:root` for theme colors.

## Testing Guidelines
Run tests with `cargo test` inside `leader-stream/` (API smoke tests). Validate changes manually by:
- Loading the page and confirming data populates and the UI updates.
- Hitting `/api/next-leaders?limit=10`, `/api/current-slot`, and
  `/api/leader-stream` (SSE) in a browser or curl.

## Commit & Pull Request Guidelines
Git history is not available in this checkout, so no established commit format
is visible. Use a clear, imperative subject line (≤72 chars), add a short body
for rationale when needed, and keep commits scoped to one change.
PRs should summarize user-visible impact, note API changes, and include a
screenshot for UI updates.

## Configuration & Security Notes
- `SOLANA_RPC_URL` overrides the default Solana mainnet RPC endpoint.
- API responses are cached with `Cache-Control`; keep responses cache-friendly.

---
> Source: [trustless-engineering/leader-stream](https://github.com/trustless-engineering/leader-stream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
