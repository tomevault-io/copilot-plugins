## rust-cobol

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
make build          # Debug build
make release        # Release build (default target)
make test           # Full workspace test suite
make lint           # clippy (-D warnings) + rustfmt check + cspell
make fmt            # Apply rustfmt
make check          # Type-check without codegen
make audit          # check + lint
make verify         # clean + audit + test
make example        # Compile & run examples/hello.cob
make install        # Install cobolc to ~/.cargo/bin
```

Single test: `cargo test --package cobol-parser -- test_parse_if`

E2E tests: `cargo test --package cobol-driver --test e2e_test -- test_name`

## Architecture

COBOL-85 through COBOL 2023 compiler. Pipeline in order:

```
Source → Lexer → Parser → Sema → HIR → C codegen → clang → native binary
```

11 crates in `crates/`, dependency order top-down:

| Crate | Role |
|---|---|
| `cobol-common` | `Span`, `FileId`, `SourceFile`, `SourceMap`, `SourceFormat`, `CobolStandard` |
| `cobol-diagnostics` | `Diagnostic` (code+message+severity+labels), `DiagnosticReporter` (accumulator) |
| `cobol-lexer` | `SourceReader` (fixed/free/variable column formats), `Lexer` → `Token` stream |
| `cobol-ast` | Untyped AST nodes: divisions, sections, statements, expressions, PICTURE |
| `cobol-parser` | Hand-written recursive descent; `Parser::parse_program() → CobolProgram` |
| `cobol-sema` | Symbol table, name resolution (qualified OF/IN), type checking, PICTURE analysis |
| `cobol-hir` | Desugared IR (e.g. EVALUATE → nested IF); `lower_to_hir()` |
| `cobol-mir` | Mid-level IR (placeholder) |
| `cobol-codegen` | Emits C source, then invokes clang/gcc to compile. Links with cobol-runtime |
| `cobol-runtime` | Rust staticlib with `extern "C"` ABI: BCD arithmetic, file I/O, string ops, intrinsics |
| `cobol-driver` | CLI (`cobolc`) orchestrating the full pipeline; clap-derive |

**Why C codegen instead of LLVM**: inkwell supports up to LLVM 18; system LLVM is 22.x. C backend is the current strategy.

## Key Patterns

**Error handling**: Parser and sema accumulate errors into `DiagnosticReporter` rather than early-returning. Functions return `Result<T, ()>` where `()` means "errors were reported." The parser crate has `#![allow(clippy::result_unit_err)]` for this reason.

**Runtime C ABI**: All runtime functions are `#[no_mangle] pub unsafe extern "C" fn` so generated C code can call them. The runtime is compiled as staticlib and linked by the codegen crate.

**COBOL source formats**: The lexer's `SourceReader` handles Fixed format (columns 1-6 sequence, column 7 indicator, columns 8-72 content area) and Free format (`*>` comments). Tests generally use Free format because Rust string literals can't easily represent fixed-format column alignment.

**Testing**: Unit tests live alongside code in `#[cfg(test)] mod tests`. E2E tests in `crates/cobol-driver/tests/e2e_test.rs` use helpers like `compile_to_hir()`, `compile_to_c()`, and `parse_and_lower()` (skips sema for cases where semantic analysis would reject test fixtures).

## Conventions

- Rust edition 2021, stable toolchain, min version 1.75
- `rustfmt.toml`: max_width = 100
- Clippy: `-D warnings` (all warnings are errors)
- `SmolStr` (from `smol_str` crate) used for token text and identifiers throughout
- License: MIT OR Apache-2.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izuno4t) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
