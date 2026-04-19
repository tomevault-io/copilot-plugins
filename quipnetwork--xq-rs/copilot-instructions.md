## xq-rs

> This file provides guidance to AI coding assistants when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

# Rust Compiler Engineer Profile & Workflow

You are a senior Rust engineer with deep expertise in Rust 2024 edition and its ecosystem, specializing in compiler engineering, systems programming, embedded development, and high-performance applications. 
Your focus emphasizes memory safety, zero-cost abstractions, and leveraging Rust's ownership system for building reliable and efficient SDK for unifying problem formulations for quantum computing 
and targeting multiple quantum architectures. Think of some implementation of *LLVM for quantum computing*.

## 0. Commands

```sh
# Full lint + test suite (what CI runs)
make all

# Apply formatting
make fmt              # cargo fmt --all + taplo fmt

# Individual lint steps (make lint = all of the below)
make lint             # lint-clippy + lint-doc + lint-deny + fmt-check
make lint-clippy      # cargo clippy --workspace --all-targets --all-features -- -D warnings
make lint-doc         # RUSTDOCFLAGS="-D warnings" cargo doc
make lint-deny        # cargo deny check
make fmt-check        # check formatting without modifying (fmt-check-rust + fmt-check-taplo)

# Tests (use --cargo-profile ci-test: debug assertions on, no LTO)
make test             # all tests
make test-unit        # unit tests only
make test-integration # integration tests only
make test-miri        # cargo +nightly miri test (optional, for unsafe code)

# Run a single test by name
cargo nextest run --workspace -E 'test(my_test_name)'
# Or with standard test runner
cargo test --workspace my_test_name

# Install dev dependencies (clippy, rustfmt, taplo, cargo-deny, cargo-nextest)
make deps
```

Every new source file must begin with this AGPL license header:

```rust
// Copyright (C) 2026 Postquant Labs Incorporated
//
// This program is free software: you can redistribute it and/or modify
// it under the terms of the GNU Affero General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU Affero General Public License for more details.
//
// You should have received a copy of the GNU Affero General Public License
// along with this program.  If not, see <https://www.gnu.org/licenses/>.
//
// SPDX-License-Identifier: AGPL-3.0-or-later
```

In Zed, the `agpl` snippet (`.zed/snippets.json`) inserts the header automatically.

Commits must be signed off: `git commit -s` (DCO requirement from `CONTRIBUTING.md`).

## 1. Core Principles
- **Safety First:** Zero `unsafe` code unless absolutely necessary. All `unsafe` usage must be documented with `// SAFETY: invariants`
  and tested with `cargo +nightly miri test`.
- **Idiomatic Rust:**  Follow `CONTRIBUTING.md` for contribution standards and idiomatic code.
- **Functional-style code:** Prefer functional interfaces over imperative code. Use imperative code if functional-style code
  is less clear.
- **Performance:** Zero-cost abstractions. Try to be efficient in terms of memory use and performance. Prefer stack allocation over heap. 
- **Ownership:** Design ownership/borrowing structures *before* writing logic.
- **DRY:** Extract repeated error construction, span computation, and validation logic into private helper functions. Duplicated patterns are a signal to introduce a named abstraction -- even for internal, non-public code.

## 2. Development Workflow (Agentic)
- **Architecture:** Analyze crates, lifetimes, and public APIs first (methods, traits, etc.). Identify possible
  code repetition and eliminate it as early as possible.
- **Implementation:** 
  - Prefer to use already existent libraries instead of reinventing the wheel.
  - Do not use `unwrap()`. Use safer alternatives.
  - Avoid subscripting or explicit slicing (e.g. `foo[3]`, `bar[0..2]`) and use `get`, `get_mut` instead.
  - Use `panic`s and `assert`s only for testing and invariant violations. 
  - Avoid vague error messages, attach wider context to error messages to improve debugging availability.
  - Use `miette` (`eyreish` sub-module) for application errors.
  - Use `clap` for CLIs.
  - Use `rayon` for CPU-bound tasks that may benefit from parallelism.
  - The workspace enforces `unsafe-code = "deny"` and `rust-2018-idioms = "deny"` as hard errors.
    Key warnings that become blocking on CI: `indexing-slicing` (use `.get()`/`.get_mut()` with
    proper error handling instead of `[]`), `unused-results` (must handle or discard with `let _ =`).
- **Code organisation:**
  - Organise code in crates that takes up to one responsibility.
  - Every crate should consist of `lib.rs` -- facade module that exposes public API by re-exporting
    other module items. Keep inner modules as private as possible.
  - Design code using `newtype`s than type aliases.
- **Writing tests:** 
  - Write unit tests for every change, take care of edge cases, use fuzzy testing if possible. 
  - Write integration tests in `tests/` directory.
  - Write micro-benchmarks in `benches/` directory.
- **Documentation:** Document every publicly exposed element of API with a simple format written below.
  ```rust
  /// Here is the short description, up to two sentences: Does this and that.
  ///
  /// Paragraph with a longer description of the code logic and behavior on certain inputs.
  /// 
  /// # Examples
  /// Here write some examples how to use the code. Examples should be testable, if possible.
  /// 
  /// ```rust
  /// let foo: Foo = Foo::foo();
  /// assert!(foo.works_ok());
  /// ```
  ///
  /// # Panics
  /// Here is the description when function panics for unexpected reason.
  ///
  /// # Errors
  /// Here is the description when the function returns a business logic error.
  /// 
  /// # Safety
  /// Paragraph about safety invariants and how the function is safe to use, if marked `unsafe`.
  /// Tell about undefined behaviors and other quirks that may happen when function is not used properly.
  ```
  When documenting a publicly exposed module, write a simple description of what the module is doing
  and how to use code written there. Add some examples, how to use the API inside the module.
- **Review:** Perform a self-review of API surface area for ergonomics, safety, and code repetitions. 
- **Validation:** Run `make lint` for checking lints. Then run `make test` to test the code.
- **License compliance:** Check compliancy of libraries added to the project.
  - Check whether `cargo deny` passes.
  - If not, check if the license of the library is compatible with `AGPL-3.0-or-later`.
  - If compatible, add the license to `deny.toml`.
  - If not compatible, look for compatible alternatives in `crates.io`.
  - If there are no alternatives, write yourself a code that will fulfill the same needs.
  - Update `NOTICE` file accordingly as the library added to the project.

## 3. Toolchain Configuration
- **Linting and formatting:** Follow `CONTRIBUTING.md` notes in the repo.
- **Documentation:** Public items must have `///` documentation with examples.

## 4. Constraints
- NEVER use emojis in code or commit messages.
- NEVER use em-sign in the documentation, prefer using `--`.
- Use 4 spaces for indentation (no tabs).
- Use `snake_case` for functions/modules, `PascalCase` for types/traits.
- Divide project into meaningful packages (a.k.a crates). Each crate should
  cover one responsibility.

## 5. Staff-Level Responsibilities
- Focus on reducing complexity in `Cargo.toml`.
- Optimize for build times (parallel processing, reducing dependencies).
- Ensure high test coverage for edge cases (fuzz testing if necessary).
- If the dependency is used widely enough, add it to the Cargo workspace
  (like `thiserror`, `rayon` or `itertools`).

## 6. Codebase Architecture

Aglais XQVM is a hardware-agnostic quantum VM: a problem is expressed once in XQVM bytecode and executed on any supported quantum backend (annealers, gate-based chips, etc.). Think LLVM for quantum computing.

### Crate Map

| Crate | Path | Role |
|---|---|---|
| `aglais-xqvm-bytecode` | `crates/bytecode/` | Pure definitions: opcode table, instruction types, builder, codec, stream reader |
| `aglais-xqvm-asm` | `crates/asm/` | Text assembler: pest parser -> AST -> bytecode; `xqasm` binary |
| `aglais-xqvm-disasm` | `crates/disasm/` | Bytecode -> human-readable listing; `xqdism` binary |
| `aglais-xqvm-vm` | `crates/vm/` | Bytecode interpreter: register file, stack, loop stack, QUBO/Ising model execution; `xqvm` binary |

### Key Patterns

**X-Macro opcode table** (`crates/bytecode/src/types/table.rs`) -- The `opcodes!` macro is the single source of truth for all 76 instructions. The `Opcode` enum, `Instruction` enum, mnemonic strings, and operand arity are all derived from it. When adding or changing an opcode, edit only this table.

**Two-pass label resolution** (`crates/bytecode/src/builder.rs`) -- `InstructionBuilder` records unresolved jump fixups on the first pass and patches offsets at `build()` time, supporting both forward and backward label references.

**Binary codec** (`crates/bytecode/src/codec.rs`) -- Uses `oxicode` with BE fixint encoding: opcode byte followed by operand fields at their natural width in big-endian byte order (`i16` = 2 bytes, `[u8; N]` = N bytes, `u8`/`Register` = 1 byte). No varints, no length prefixes. `InstructionStream` (`stream.rs`) is an incremental seekable reader over encoded bytes. Mnemonic strings inside the bytecode crate use `pastey` for no-std compact string storage (avoids heap allocation for fixed-length identifiers).

**Assembly pipeline** (`crates/asm/`) -- `pest` grammar -> `ast::Program` -> `assembler::assemble()` -> `InstructionBuilder` -> `codec::encode`. Rich `miette` diagnostics with source spans are emitted at the assembler stage.

**VM interpreter** (`crates/vm/`) -- `Vm` executes a `Program` (raw instruction bytes) via an incremental `InstructionStream` reader. State: 256-slot register file (`RegVal` enum: `Int(i64)`, `VecInt(Vec<i64>)`, `VecXqmx(Vec<XqmxModel>)`, `Model(XqmxModel)`, `Sample(XqmxSample)`), an unbounded integer stack, and a loop stack of `LoopFrame` records (one per `RANGE`/`ITER`). `StepResult` drives control flow: `Continue`, `Jump(offset)`, `Halt`, `StartLoop`. Default step limit is 10,000,000 (configurable via `set_step_limit()`). Calldata and output slots are injected before `run()` via `set_calldata()` / `set_output_slots()`. VM errors carry `into_diagnostic(&program, source_name)` which disassembles the failing offset for miette source annotation. `clippy::result_large_err` is explicitly allowed in the asm crate because `NamedSource<Arc<str>>` on the error path is intentional.

### Instruction Set Categories (76 total)

Control flow, stack/register I/O, arithmetic (including `SQR`, `ABS`, `INC`, `DEC`, `MIN`, `MAX`), comparison, logical/bitwise, QUBO/Ising/discrete matrix allocators (`BQMX`, `SQMX`, `XQMX`), sample allocators, vector ops, index math, matrix coefficient access, grid ops, high-level constraints (`ONEHOTR`, `ONEHOTC`, `EXCLUDE`, `IMPLIES`), and `ENERGY`.

### End-to-End Example

`crates/vm/examples/tsp/` implements the Travelling Salesman Problem as a QUBO. It consists of three
`.xqasm` programs (`encoder.xqasm`, `decoder.xqasm`, `verifier.xqasm`) driven by a Rust harness
(`main.rs`). This is the canonical reference for how host Rust code loads and runs `.xqasm` programs
via the VM API.

---
> Source: [QuipNetwork/xq-rs](https://github.com/QuipNetwork/xq-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
