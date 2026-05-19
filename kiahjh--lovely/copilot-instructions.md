## lovely

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lovely is a low-level systems programming language designed for creating the Humble operating system. It's a compiled, type-driven language with a focus on safety and performance.

## Build Commands

### Build the Compiler
```bash
cargo build --release
```

### Install the Compiler Locally
```bash
cargo install --path .
# or use justfile
just install
```

### Build a Lovely Program
```bash
lovely build <path-to-project-directory>
```

The build command expects a directory containing:
- A `config.toml` file with a `package = "name"` field
- One or more `.lv` source files

Example:
```bash
lovely build examples/simple_example
```

### View Assembly Output
```bash
lovely build <path> --asm
```

### Run Tests
```bash
cargo test
```

## Compiler Architecture

The compiler follows a traditional pipeline architecture:

1. **Lexer** (`src/lexer/`) - Tokenizes source code
2. **Parser** (`src/parser/`) - Produces an Abstract Syntax Tree (AST)
3. **Combiner** (`src/blush/combiner.rs`) - Combines multiple source files into a single AST
4. **Checker** (`src/checker/`) - Type checking and semantic analysis
5. **IR Generator** (`src/ir/`) - Generates Three-Address Code (TAC) intermediate representation
6. **Code Generator** (`src/codegen/`) - Emits assembly code for target platforms

### Key Pipeline Flow (src/blush/mod.rs)

The `Blush::build()` function orchestrates the entire compilation:

1. Collects all `.lv` files from the project directory
2. Parses each file into an AST
3. Combines all ASTs using the Combiner
4. Type checks the combined program
5. Generates IR from the checked program
6. Generates assembly from IR
7. Assembles with `nasm` and links with `ld` to produce executable

### Type System (src/checker/)

- Built-in types: `Int`, `Bool`, `Unit` (stored as constants INT_ID, BOOL_ID, UNIT_ID)
- Function types with parameter modifiers: `read` (immutable borrow), `mut` (mutable borrow), `take` (ownership transfer)
- Scoped type and variable resolution with shadowing support
- Type hints flow through expressions for better inference

### IR Layer (src/ir/)

- Uses Three-Address Code (TAC) representation in `src/ir/tac.rs`
- Organizes code into Basic Blocks with labels
- Special handling for the `main` function (generates `Exit` instruction instead of `Ret`)
- Variables are uniquely identified with `name#id` format to handle shadowing

### Code Generation (src/codegen/emitters/)

Currently supports:
- `x86_64_linux_nasm` - Primary backend for x86-64 Linux using NASM syntax
- `aarch64_apple_gas` - Partial support for Apple Silicon using GAS syntax

Target definitions in `src/targets/mod.rs`.

## Project Structure

The build system generates a `build/` directory in the project folder containing intermediate files and the final executable (named after the package).

## Language Syntax

See `syntax.lv` for language reference. Key features:
- Variable declarations: `name: Type: value` (immutable) or `name: Type = value` (mutable)
- Functions: `fun(params) -> Type: body`
- Function parameters can be labeled at call sites
- Namespace support with `#` operator (e.g., `module#function`)
- Comments: `--` for line comments

## Development Notes

- The codebase uses extensive Clippy lints (pedantic, nursery, perf, correctness, suspicious, style, complexity, cargo)
- Testing framework uses `insta` for snapshot testing (though tests are currently being set up)
- The `FileTreeNode` type in `src/blush/mod.rs` is used to recursively traverse and transform project files

---
> Source: [kiahjh/lovely](https://github.com/kiahjh/lovely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
