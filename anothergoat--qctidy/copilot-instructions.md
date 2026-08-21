## qctidy

> You are an expert writing code for this project.

# Documentation for code agents

You are an expert writing code for this project.

## Your role

- You are fluent in Markdown, Rust, Python and TOML.
- You write for developers. Focus on clarity, readability and conciseness.
- You may update or generate documentation in `docs/` and read or write code in `crates/`.

## Project overview

- The project uses Rust 2024 edition.
- Important dependencies are:
  - `getset`, `inew` and `strum` for useful macros.
  - `thiserror` for error handling.
  - `faer` for linear algebra.
  - `serde` for parsing and serialization.
  - `pyo3` for Python bindings.
- Inside `crates/`, you will find these subcrates:
  - `analyzer`: Circuit analysis and metrics (library).
  - `cli`: Command-line interface (binary).
  - `codegen`: Code generator (library).
  - `converter`: Circuit parser and serializer for various formats (library).
  - `estimator`: Circuit execution time and cost estimation (library).
  - `facade`: Facade for client implementation (library).
  - `library`: The core of QCTidy (library).
  - `ports`: Ports for optional services (library).
  - `presenter`: Circuit and graph visualization (library).
  - `qiskit`: Python + Qiskit bindings via PyO3 (library).
  - `server`: Server implementation as a REST API (binary).

## Useful commands you can use

- `cargo check --workspace`: Check that all the crates compile.
- `cargo clippy --workspace`: Lint code in all the crates.
- `cargo test -p <crate>`: Run tests for a specific crate.
- `RUST_BACKTRACE=1 cargo test  -p <crate>`: Run tests for a specific crate, with backtrace.
- `tree --gitignore` to see the project structure (or variations of `tree`).

Also take a look at the [justfile](justfile) for more references.

## Documentation

- Be concise and specific.
- Write so that even a new developer can understand. Don't assume that they are experienced in what you're talking about.

## Code in general

- Avoid adding any comments stating the obvious.
- Don't use short variable names, prefer descriptive names (example: prefer `column` instead of `col`).

## Rust code

- Follow Rust conventions.
- Prefer using existing dependencies instead of adding new ones that do the same thing.
- Use `unwrap()` for cases where you expect a value to be present, don't add `expect()` yourself.

## Version control

- Don't add files to `.gitignore`.
- Don't modify files that are ignored by Git.

## Boundaries

- Do without asking:
  - Write to `docs/` or `crates/`.
  - Anything you're explicitly told to do.
- Ask first before:
  - Before adding or changing dependencies.
  - Before moving or deleting files.
  - Before writing to directories that are not `docs/` or `crates/`.
- Never do:
  - Make Git commits of any kind.

---
> Source: [AnotherGoat/qctidy](https://github.com/AnotherGoat/qctidy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
