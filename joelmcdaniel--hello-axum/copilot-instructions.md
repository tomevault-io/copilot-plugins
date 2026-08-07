## hello-axum

> A minimal Rust web API built with Axum.

# hello-axum

A minimal Rust web API built with Axum.

## Stack
- Rust 2024 edition
- axum 0.8.x
- tokio (multi-thread runtime)

## Commands
- `cargo run` — start the server on http://127.0.0.1:3000
- `cargo check` — fast type check
- `cargo clippy -- -D warnings` — lint; must pass clean
- `cargo fmt` — format; run before committing
- `cargo test` — run tests

## Layout
- `src/main.rs` — entrypoint, router setup, handlers

## Endpoints
- `GET /hello` → `Hello, World!`
- `GET /hello/{name}` → `Hello, <name>!`

## Conventions
- Axum 0.8 uses `{param}` path syntax, not `:param`.
- Handlers are `async fn` returning something implementing `IntoResponse`.
- Prefer extractors (`Path`, `Query`, `Json`) over manual request parsing.
- Extractors consuming the body must come last in the handler signature.
- Return `Result<T, AppError>` for fallible handlers rather than panicking.
- Keep `main` thin: as routes grow, move handlers into `src/routes/`.

---
> Source: [joelmcdaniel/hello-axum](https://github.com/joelmcdaniel/hello-axum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
