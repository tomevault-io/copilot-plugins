## aam-rs

> AAM is a robust, lightweight configuration parser written in Rust. It supports key-value pairs, recursive dependency resolution (`find_deep`), file imports (`@import`), bidirectional lookups (`find_obj`), and schema validation. The project uses a custom `.aam` format.

# AAM (Abstract Alias Mapping) - Project Context

## Project Overview
AAM is a robust, lightweight configuration parser written in Rust. It supports key-value pairs, recursive dependency resolution (`find_deep`), file imports (`@import`), bidirectional lookups (`find_obj`), and schema validation. The project uses a custom `.aam` format.

It is designed with a core Rust library (`aam-rs`) and provides bindings for multiple languages including C#, Node.js, Python, Ruby, PHP, Java, Go, and WebAssembly.

Recently, the codebase has transitioned to a new "pipeline" architecture located in `src/pipeline/` which implements strict Separation of Concerns (Lexer -> Parser -> Validator -> Executer).

## Building and Running
The core project is built and tested using standard Cargo commands:
- **Build:** `cargo build`
- **Test:** `cargo test`

The repository also includes separate directories for language bindings (e.g., `csharp/`, `js/`, `python/`, `go/`, `java/`, etc.), each using their respective build systems (e.g., `dotnet`, `npm`, `gradle`).

## Development Conventions
- **New Architecture Pipeline:** The parsing logic is split into `Lexer`, `Parser`, `Validator`, and `Executer` stages, driven by a `Pipeline` struct. This replaces the monolithic logic previously found in `src/aaml/`.
- **Error Handling:** Uses `AamlError` for typed, detailed parsing and I/O error handling.
- **Traits:** The pipeline components are defined by traits (e.g., `Lexer`, `ParserExecutor`, `ValidateExecutor`, `Executer`) to allow custom implementations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/INiNiDS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
