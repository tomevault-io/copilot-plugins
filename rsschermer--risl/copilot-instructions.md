## risl

> Welcome, LLM Agent. This document provides a high-level "mental map" and operational guidelines for

# RISL Agent Handbook (AGENTS.md)

Welcome, LLM Agent. This document provides a high-level "mental map" and operational guidelines for
the RISL (Rust-Integrated Shading Language) compiler project.

## Project Mental Map

RISL is a compiler for a domain-specific language integrated into Rust, targeting modern GPU APIs
(via the `empa` crate). It translates Rust code into target shader languages, primarily WGSL (the
WebGPU Shading Language) and SPIR-V. To achieve this, Rust's Mid-level Intermediate Representation 
(MIR) is lowered through 3 additional intermediate representations that are collectively referred 
to as the Shading Language Intermediate Representation (SLIR):

- **SLIR-CFG**: a Control-Flow Graph representation that is meant to be easy to target from Rust 
  MIR.
- **SLIR-RVSDG**: a Regionalized Value-State Dependence Graph that acts as the main workhorse of the
  compiler. This is where most of this compiler's code transformations are applied.
- **SLIR-SCF**: a Structured Control-Flow representation that is semantically close to both WGSL and
  SPIR-V. This representation facilitates the final translation step into the target shader language.

### Core Crates

- **`crates/compiler/rislc`**: The main compiler binary. It acts as a `rustc` wrapper and driver for
  the compilation pipeline.
- **`crates/compiler/slir`**: Defines the Shading Language Intermediate Representation.
- **`crates/library/risl*`**: Support libraries and macros that users use to write RISL code.
- **`crates/rvsdg-dump-cli`**: A diagnostic tool for analyzing RVSDG binary dumps.
  **Use this instead of raw binary inspection.**
- **`xtask`**: A dedicated crate for project automation (running tests, rebuilding the compiler).

## Operational Workflow

### Behavioral Tests

This project defines a suite of behavioral tests that verify the correctness of the compiler's
output. When making changes to the compilation-chain that may affect the output, these tests should
be run before declaring the task as complete. Because these tests need to be run with a fresh
build of the compiler, this project provides a "Cargo xtask" that combines rebuilding the compiler
and using this fresh compiler build to run the test-suite:

```bash
cargo xtask run-behavioral-tests
```

#### Running Specific Tests

Each behavioral test (or group of tests) is behind a Cargo feature. Using features allows you to
run specific tests and ensures that only the relevant shader modules are compiled. This is
particularly useful when debugging, as it limits the output (like RVSDG dumps) to the specific
module you are interested in.

To run a specific test, use the `--features` (or `-f`) flag with the xtask command:

```bash
cargo xtask run-behavioral-tests --features <feature_name>
```

For example:

```bash
cargo xtask run-behavioral-tests --features test_enum_result
```

Refer to the `[features]` section in `tests/behavioral-tests/Cargo.toml` for a list of available
features.

### RVSDG Analysis

When debugging compiler transformations or structural issues in the RVSDG you may wish to inspect
the RVSDG's structure. To facilitate this the `slir::rvsdg::Rvsdg` data structure provides a 
`dump_to_file` method that can be used to serialize the RVSDG to a "dump" file. If you believe 
that knowledge of the RVSDG structure can help you debug problems, such as panics and incorrect
output, then you are encouraged to insert `dump_to_file` statements in the project at points of
interest. For example, if a panic occurs during a SLIR transformation, you may want to insert a
`dump_to_file` statement right before the code that triggers the panic. Note that you can insert
multiple `dump_to_file` statements at different points in the code that each dump to a different
file. Such `dump_to_file` statements are only meant to be temporary debugging tools and should be
removed once the problem is resolved.

This project also provides the [rvsdg-dump-cli](crates/rvsdg-dump-cli) tool to facilitate 
inspecting such "dump" files. This tool can be used to write a textual representation of (parts of)
the RVSDG to the terminal. You should read the [README](crates/rvsdg-dump-cli/README.md) for this
tool to help you understand how to use it and interpret the output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RSSchermer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RSSchermer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
