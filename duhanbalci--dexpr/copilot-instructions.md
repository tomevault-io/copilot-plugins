## dexpr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

dexpr is an embeddable expression evaluator and bytecode VM written in Rust. It parses expressions and simple scripts (`.dexpr` files), compiles to bytecode, and executes on a register-based virtual machine. Designed to be embedded in other projects as a rule engine, formula evaluator, or expression calculator.

## Common Commands

```bash
# Run tests
cargo test

# Run benchmarks
cargo bench --bench my_benchmark

# Run the main program (executes basic_long.dexpr by default)
cargo run --release

# Profile with samply
cargo build --profile profiling
samply record ./target/profiling/dexpr
```

## Architecture

The pipeline flows: **Parser → AST → Compiler → Bytecode → VM**

### Core Modules

- **`parser/`**: PEG-based parser (`grammar.rs`) that produces AST nodes
- **`ast/`**: Expression (`expr.rs`), statement (`stmt.rs`), and value (`value.rs`) types
- **`compiler.rs`**: Transforms AST into bytecode; single-pass, globals-only, register allocation
- **`bytecode.rs`**: `BytecodeWriter` for compilation, `BytecodeReader` for VM consumption
- **`opcodes.rs`**: Defines `OpCodeByte` enum and `Register` type
- **`vm/vm.rs`**: Register-based VM with global variable support; executes bytecode

### Key Types

- `Value` (ast/value.rs): Runtime values (Number using rust_decimal, String, Boolean, Object using indexmap, etc.)
- `Compiler`: Stateful compiler with register allocation
- `VM<'a>`: Bytecode interpreter with global variable support via `set_global`/`get_global`

### Typical Usage Pattern

```rust
use dexpr::{ast::value::Value, compiler::Compiler, parser, vm::VM};

let ast = parser::program(source_code)?;
let mut compiler = Compiler::new();
let bytecode = compiler.compile(ast)?;
let mut vm = VM::new(&bytecode);
vm.set_global("input", Value::Number(dec!(42)));
vm.register_function("getRate", |args| Ok(Value::Number(dec!(34.5))));
let result = vm.execute()?; // Returns last expression's value
// Or use globals: let output = vm.get_global("output");
```

## Language Features

The dexpr language supports:
- If/else conditionals with `if ... then ... else ... end`
- String methods (e.g., `.upper()`, `.lower()`, `.trim()`, `.trimStart()`, `.trimEnd()`, `.split()`, `.replace()`, `.contains()`, `.startsWith()`, `.endsWith()`, `.length`, `.charAt()`, `.substring()`)
- Arithmetic (`+`, `-`, `*`, `/`, `%`, `**`), comparison, and logical operators
- `in` operator for membership testing (`"finans" in categories`, `5 in numbers`, `"hello" in "hello world"`, `"key" in obj`)
- Compound assignments (`+=`, `-=`, `*=`, `/=`, `%=`)
- Built-in `log()` function for output and `rand(min, max)` for random integers
- External (host) function registration via `vm.register_function()`
- External (host) method registration via `vm.register_method()`
- Expression return value: `execute()` returns the last expression's value
- Line comments (`//`) and block comments (`/* */`)
- Lists: `NumberList` and `StringList` types with methods (`sum`, `avg`, `min`, `max`, `first`, `last`, `get`, `join`, `contains`, `indexOf`, `slice`, `reverse`, `sort`, `isEmpty`, etc.)
- Objects: `Object` type (provided externally via `set_global`) with property access (`obj.field`), nested access (`obj.a.b`), property assignment (`obj.field = value`), and methods (`keys()`, `values()`, `length()`, `contains(key)`, `get(key)`)
- Lists: `List` type for heterogeneous arrays (including array of objects), with methods (`length`, `isEmpty`, `first`, `last`, `get`, `contains`, `indexOf`, `slice`, `reverse`, `join`, `map(field)`, `filter(field, value?)`, `find(field, value?)`, `sort(field)`). Property projection: `kalemler.tutar` extracts field from each Object element, returning NumberList/StringList/List. `map("field")` also available as explicit alternative

## Detailed Module Documentation

**IMPORTANT:** Before making any changes to the codebase, read the relevant documentation files in the `docs/` folder to understand how that module works in detail. After making changes to any module, update the corresponding documentation file to keep it in sync.

- **[docs/architecture.md](docs/architecture.md)** — Overall architecture, pipeline, design decisions, dependencies
- **[docs/ast.md](docs/ast.md)** — AST module: Expr, Stmt, Value types, Span, serialization format
- **[docs/parser.md](docs/parser.md)** — Parser: PEG grammar rules, operator precedence, reserved keywords
- **[docs/opcodes.md](docs/opcodes.md)** — Opcodes: full instruction set with hex values and categories
- **[docs/bytecode.md](docs/bytecode.md)** — Bytecode: BytecodeWriter/Reader API, data format, disassembler
- **[docs/compiler.md](docs/compiler.md)** — Compiler: single-pass compilation, register allocation, label/jump system
- **[docs/vm.md](docs/vm.md)** — VM: execution loop, opcode handlers, error types, DebugInfo
- **[docs/language_info.md](docs/language_info.md)** — Language Info: editor metadata generation, JSON format, host integration
- **[docs/editor.md](docs/editor.md)** — Editor: CodeMirror 6 language support, Lezer grammar, type-aware autocomplete

**Rule:** Any code change that modifies the behavior, API, or structure of a module MUST be accompanied by an update to the corresponding `docs/` file. Documentation and code must always stay in sync.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duhanbalci)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/duhanbalci)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
