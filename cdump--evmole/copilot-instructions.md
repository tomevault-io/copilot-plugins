## evmole

> EVMole extracts structured facts from deployed EVM runtime bytecode.

# EVMole agent guide

EVMole extracts structured facts from deployed EVM runtime bytecode.

- This is the EVMole implementation repository. For requests to implement,
  debug, refactor, review, or test EVMole itself, inspect and modify the
  repository source. Do not use the published EVMole packages, MCP server,
  JSON CLI, or portable skill as a substitute for understanding the
  implementation. Running locally built libraries, binaries, and adapters for
  validation is expected.
- Treat input as deployed/runtime bytecode. Creation bytecode is not executed or
  stripped automatically.
- Route repository development by component:
  - Core analysis algorithms and the public Rust API: `src/` and `Cargo.toml`.
  - Python bindings: `src/interface_py.rs` and `python/`.
  - JavaScript/WASM bindings and the JSON CLI: `src/interface_js.rs`,
    `src/interface_wasm.rs`, and `javascript/`.
  - Go bindings: `go/`.
  - MCP server, shared agent adapter, portable skill, schemas, and agent tests:
    `javascript/src/agent_api.mjs` and `agent/`.
- Use `agent/skills/evm-bytecode-analysis/SKILL.md` only when the user asks to
  analyze supplied deployed runtime bytecode, or when changing or testing the
  CLI, MCP server, portable skill, schemas, interpretation behavior, or agent
  integration.
- When helping a separate project integrate EVMole, use the existing binding
  for the project's language: the `evmole` Rust crate, `evmole` JavaScript
  package, `evmole` Python package, or `github.com/cdump/evmole/go`. Start with
  `README.md` and the language-specific README.
- Reuse the shared agent adapter when changing the CLI, MCP server, skill, or
  agent tests; do not duplicate its request, response, validation, or
  pagination logic.
- Validate the components changed:
  - Rust: `cargo fmt --check`, `cargo test`, and
    `cargo clippy --all-features -- -D warnings`.
  - JavaScript and the agent adapter: build with
    `npm --prefix javascript run build`, then run focused tests with
    `npm --prefix javascript test` and `npm --prefix agent/mcp test`.
  - Go: build the embedded WASM with `make -C go wasm`, then run
    `make -C go test`.
  - Python: build with Maturin and run `python3 python/test_python.py`; use the
    release workflow for the supported Python and platform matrix.
- When reporting bytecode-analysis results, describe inferred arguments,
  mutability, and storage information as inferred, not verified source-level
  facts.
- Preserve unrelated benchmark providers, datasets, result directories, and
  user changes.

---
> Source: [cdump/evmole](https://github.com/cdump/evmole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
