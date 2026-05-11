## rthagore

> - `src/` holds the Rust compiler and CLI. Key areas include `src/lexer/`, `src/parser/`, `src/types/`, `src/analysis/`, `src/mir/`, `src/backend/`, and `src/diagnostics/`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds the Rust compiler and CLI. Key areas include `src/lexer/`, `src/parser/`, `src/types/`, `src/analysis/`, `src/mir/`, `src/backend/`, and `src/diagnostics/`.
- `tests/` contains integration tests plus fixtures in `tests/data/`, `tests/integration/`, and `tests/wasmhostruntime/`.
- `docs/` is canonical documentation; `examples/` hosts Thagore samples; `stdlib/` is the standard library; `extension/` is editor tooling; `bootstrap/` holds seed assets.
- Constraints: do not remove `tests/`, `docs/`, `extension/`, or `examples/`. Avoid changes that alter compiler core behavior without explicit agreement.
- Module layout: keep files small and cohesive, prefer folder-based modules, and avoid `include!` inside function bodies. Wasm runtime code lives under `src/backend/runtime/wasm/` with exec helpers in `src/backend/runtime/wasm/exec/`. New file names should be descriptive and avoid `_` or `-` unless matching an existing pattern in that folder.

## Build, Test, and Development Commands
- `cargo build` compiles the compiler and CLI.
- `cargo run -- run examples/hello.tg` runs a sample program.
- `cargo run -- check examples/hello.tg` parses and type-checks without execution.
- `cargo test` runs the Rust test suite (the CLI binary is excluded by default).
- Optional: `cargo fmt` and `cargo clippy` for formatting and linting.

## Coding Style & Naming Conventions
- Rust edition 2024.
- Use `ZenResult` and `ZenError` for error paths.
- Use `log` + `env_logger`; avoid `println!` in library-style code.
- Follow rustfmt defaults; keep modules focused and readable.

## Testing Guidelines
- Prefer tests in `tests/` for language and backend behavior; add module unit tests when appropriate.
- Name tests after the behavior or feature; keep fixtures under `tests/data/`.

## Commit & Pull Request Guidelines
- Commit messages in history are short, descriptive, and often start with a verb; some use Conventional Commit prefixes like `feat:` or `fix:`. Follow that pattern and keep one logical change per commit.
- For PRs: open an issue first for syntax or language changes, include a concise summary, rationale, and tests; update `docs/` and `README.md` when behavior changes. Include screenshots for extension or UI updates.

## Security & Reporting
- Follow `CODE_OF_CONDUCT.md` and `SECURITY.md` for reporting issues or vulnerabilities.

---
> Source: [thagore-foundation/RThagore](https://github.com/thagore-foundation/RThagore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
