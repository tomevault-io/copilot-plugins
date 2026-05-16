## ast-serialize

> This repository contains a Python Rust extension that serializes a Ruff AST into binary data.

# Overview

This repository contains a Python Rust extension that serializes a Ruff AST into binary data.
Mypy uses this extension to parse Python code and deserialize the produced AST.

This depends on Ruff crates as external git dependencies.

# Code structure

- Serialization is implemented in ~/src/ast_serialize/src/serialize_ast.rs (and other files in src/)
- Deserialization is implemented in ~/src/mypy/mypy/nativeparse.py
- Mypy AST is defined in ~/src/mypy/mypy/nodes.py (type annotation AST nodes in ~/src/mypy/mypy/types.py)

## Ruff sources

This project depends on Ruff crates as git dependencies (see Cargo.toml). The sources are:

- **Git dependency**: Defined in Cargo.toml, pinned to a specific Ruff tag/commit
- **Cached location**: After first build, sources are cached in `~/.cargo/git/checkouts/ruff-*/`
- **Upstream repo**: https://github.com/astral-sh/ruff

To browse Ruff sources:
```bash
# Find the cached checkout
ls ~/.cargo/git/checkouts/ruff-*/

# Or clone separately
git clone https://github.com/astral-sh/ruff ~/src/ruff
```

Key Ruff crates used:
- `ruff_python_parser` - Python parser
- `ruff_python_ast` - Python AST definitions
- `ruff_source_file` - Source file handling (LineIndex)
- `ruff_text_size` - Text position types (TextRange, Ranged)

# Testing and development

- Run `maturin develop` in ~/src/ast_serialize before running Python tests if Rust code has been modified
- Add end-to-end tests to ~/src/mypy/test-data/unit/native-parser.test. Look at parse.test in the same directory for the correct output format and how different AST nodes are output
- If pytest is not installed, enable virtualenv in ~/venv/mypy to run tests
- Pass argument `mypy/test/test_nativeparse.py::NativeParserSuite` to pytest to run the relevant native parser tests
- Run Rust unit tests with: `cargo test`

## Updating Ruff dependency

To update to a newer Ruff version, edit Cargo.toml and change the `tag` or `rev` parameter
for all ruff_* dependencies, then rebuild.

---
> Source: [mypyc/ast_serialize](https://github.com/mypyc/ast_serialize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
