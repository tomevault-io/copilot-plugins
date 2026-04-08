## diffract

> `diffract` is an OCaml-based library and CLI tool for parsing source files using [tree-sitter](https://tree-sitter.github.io/tree-sitter/) and performing pattern matching, structural diffing, and semantic transformations (find-and-replace at the AST level).

# GEMINI.md - Project Context: diffract

## Project Overview
`diffract` is an OCaml-based library and CLI tool for parsing source files using [tree-sitter](https://tree-sitter.github.io/tree-sitter/) and performing pattern matching, structural diffing, and semantic transformations (find-and-replace at the AST level).

### Main Technologies
- **OCaml (5.4+)**: Core logic and CLI.
- **Dune**: Build system.
- **Tree-sitter**: Used for generating concrete syntax trees (CSTs) for various languages.
- **Ctypes / C FFI**: For bridging OCaml with the C-based tree-sitter library and custom C helpers (grammar language functions are statically linked, not dynamically loaded).
- **Alcotest**: For unit testing.
- **Bechamel**: For benchmarking.

### Architecture
- **`lib/`**: Core library containing the matching engine, tree representation, and FFI bindings.
  - `tree.ml`: Pure OCaml representation of the syntax tree to avoid FFI overhead during traversal.
  - `match_parse.ml`: Handles pattern parsing, including metavariables and ellipsis expansion.
  - `match_engine.ml`: Core structural matching algorithms (`strict`, `field`, `partial`).
  - `match_search.ml`: Implements search, nested pattern contexts, indexing, and formatting.
  - `match_transform.ml`: Computes and applies edits from match results (semantic patches).
  - `languages.ml`: Static registry mapping language names to grammar functions statically linked into the binary.
  - `tree_sitter_helper.c`: C wrappers to handle `TSNode` struct-by-value issues and tree-sitter memory management.
- **`bin/`**: CLI entry point (`main.ml`).
- **`grammars/`**: contains scripts and infrastructure to build language-specific grammars.
- **`tests/`**: Comprehensive test suite using Alcotest.

## Building and Running

### Setup
Before building the OCaml project, you must build the grammar libraries:
```bash
cd grammars && ./build-grammars.sh && cd ..
```

### Build Commands
- **Build project**: `dune build`
- **Run tests**: `dune test`
- **Format code**: `dune fmt`
- **Run CLI**: `dune exec diffract -- [args]` (or use the built binary in `_build/default/bin/main.exe`)

### Dependencies
Managed via `opam`. Key dependencies include `ctypes`, `ctypes-foreign`, `cmdliner`, `yojson`, and `alcotest`.

## Development Conventions

### Coding Style
- **Formatting**: Strictly enforced via `ocamlformat`. Run `dune fmt` before committing.
- **Architecture**: Follow the split between pure OCaml logic (`tree.ml`) and FFI-based nodes (`node.ml`).
- **Naming**: Module-level matching logic is prefixed with `match_` (e.g., `match_engine.ml`, `match_parse.ml`).

### Testing Practices
- New features should include tests in `tests/`.
- Tests are organized by module (e.g., `test_node.ml`, `test_match.ml`).
- The main test entry point is `tests/test_runner.ml`.

### Language Support
To add a new language:
1. Add a C wrapper in `lib/tree_sitter_helper.c` and an `external` binding + entry in `lib/languages.ml`.
2. Add copy rule and `(foreign_archives ...)` entry in `lib/dune`.
3. Add compilation commands to `grammars/build-grammars.sh` and run it.
4. Rebuild with `dune build`.
5. Add relevant test cases in `tests/test_match.ml`.

## Key Files
- `README.md`: General overview and usage examples.
- `docs/internals.md`: Deep dive into architecture and FFI implementation.
- `docs/patterns.md`: Documentation for the pattern syntax and transformation rules.
- `lib/diffract.mli`: Public API for the library.
- `dune-project`: Project metadata and dependency definitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jespera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jespera)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
