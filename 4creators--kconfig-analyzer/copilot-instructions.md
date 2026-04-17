## kconfig-analyzer

> This document (`GEMINI.md`) serves as the foundational context and instruction manual for future AI assistant sessions working on the `kconfig-analyzer` project. It captures the design philosophy, architectural decisions, and current state of the codebase.

# Gemini AI Session Handover & Architecture Guide

This document (`GEMINI.md`) serves as the foundational context and instruction manual for future AI assistant sessions working on the `kconfig-analyzer` project. It captures the design philosophy, architectural decisions, and current state of the codebase.

**CRITICAL DIRECTIVE FOR AI**: When beginning a new session, read this document carefully to understand the established patterns. Always prioritize maintaining the high test coverage (>94%) and the granular testing philosophy outlined below.

---

## 1. Project Context
`kconfig-analyzer` is a high-performance Rust CLI utility designed to parse complex `Kconfig` build systems (primarily targeting the massive Linux Kernel tree). It builds a directed graph of configuration dependencies (`depends on`) and reverse-dependencies (`select`), calculates transitive "blast radiuses", detects Strongly Connected Components (cyclic loops), and performs "What-If" analysis to determine the impact of toggling specific configurations.

---

## 2. Core Architecture & Modules

### `src/lib.rs` (The Parser Engine)
*   **Graph Construction**: Uses `petgraph::graph::DiGraph<KNode, String>`.
*   **Node Metadata**: `KNode` stores the symbol name, file location, `description` (parsed from prompt/type definitions like `bool "Prompt"`), and the full multi-line `help` text block.
*   **File Discovery**: Uses the `ignore` crate (`WalkBuilder`). It aggressively honors `.gitignore` files at the root and recursively through subdirectories. It explicitly ignores hidden folders (like `.git`) and common backup files (`~`, `.orig`, `.rej`).
*   **Intelligent Caching**: Before running the heavy Regex parser, the engine builds a `std::hash::DefaultHasher` footprint of the target repository based on exact file paths, sizes, and `mtime` (modified time). 
    *   **Cache Location**: The resulting serialized GraphML is saved to `~/.kconfig-analyzer/cache/<SAFE_REPO_PATH>/repo_TIMESTAMP_<HASH>.graphml`.
    *   **Behavior**: If the repository hasn't changed, the hash matches, and the engine instantaneously loads the GraphML cache, completely bypassing text parsing.

### `src/overlay.rs` (The "What-If" Engine)
*   **Philosophy**: We *never* mutate the base `DiGraph` when testing configurations.
*   **`ConfigOverlay`**: A memory-efficient `HashMap` that stores explicit user overrides (`ConfigState::Active` or `ConfigState::Inactive`).
*   **`ImpactAnalyzer` Trait**: Implemented on the `DiGraph`. It uses a Breadth-First Search (BFS) queue to cascade state changes through the graph.
    *   *Activation*: Propagates forward through `select` edges and backward through `depends on` edges (enabling prerequisites).
    *   *Deactivation*: Propagates forward through `depends on` (disabling dependent features) and backward through `select` (disabling parent features that require it).
*   **`what_if`**: Calculates the precise delta (number of nodes whose state changed) when modifying a single symbol.

### `src/graph_io.rs` (Serialization)
*   **GraphML**: Full XML serialization/deserialization using `quick-xml`. Safely escapes and preserves multi-line `help` texts and `description` metadata in custom `<data>` keys.
*   **DOT Format**: Plaintext serialization for Graphviz. Achieves full metadata parity with GraphML by safely escaping quotes (`\"`) and newlines (`\n`) directly into the DOT `[label="..."]` attribute. Custom Regex state machine used for DOT deserialization.

### `src/scc.rs` (Cycle Detection)
*   Identifies mutually recursive dependency loops. Supports both `Tarjan's Algorithm` and `Gabow's Path-Based Algorithm`.

---

## 3. Testing Philosophy & Guidelines

The project maintains a rigorous, fully automated test suite. **Never write monolithic tests.**

### Test Organization
1.  **`tests/cli_tests.rs`**: Tests CLI argument parsing, expected `stderr` failures, and standard console output formatting via `std::process::Command`.
2.  **`tests/integration_tests.rs`**: Tests internal library logic.
    *   **Synthetic Data**: Tests must dynamically create temporary directories (`setup_integration_data`), write exact, minimal `Kconfig` mock files, run assertions, and clean up. Never rely on static repo files for unit logic.
    *   **Coverage**: Tests caching hits/misses, regex parsing edge-cases, and Gitignore adherence (up to 5 layers deep).
3.  **`tests/import_tests.rs`**: Proves mathematical roundtrip safety. Generates a graph, exports to GraphML/DOT, imports back, and asserts 0% data loss.
4.  **`tests/overlay_tests.rs`**: Verifies the complex BFS forward/backward propagation of the `ImpactAnalyzer`.
5.  **`tests/linux_roundtrip_tests.rs`**: **HEAVY STRESS TESTS**. Marked with `#[ignore]`. These tests look for a real `../linux` repository. If found, they parse all 20,000+ nodes, export/import them, run SCC algorithms, and execute randomized What-If analyses to ensure the engine doesn't panic under massive load.

### Code Coverage Execution
We use `cargo-llvm-cov`. Always run the test runner scripts to verify your changes haven't dropped coverage:
*   **Windows**: `powershell.exe -ExecutionPolicy Bypass -File .\run_tests.ps1`
*   **Linux/macOS**: `./run_tests.sh`
*   *Note*: The scripts automatically set `CARGO_INCREMENTAL=0` to prevent LLVM lockfile collisions. They output results to `tests/COVERAGE.md`.

---

## 4. Current Status & Future Next Steps

**Current Status**: The core parsing, caching, SCC detection, metadata extraction, GraphML/DOT I/O, and in-memory What-If BFS cascading logic are **100% complete and heavily tested**.

**Next Logical Steps for Future Sessions**:
1.  **Config File Ingestion**: The `ConfigOverlay` currently has stubs (`load_from_file`). Implement the logic to parse a standard Linux `.config` file (lines like `CONFIG_FOO=y` or `# CONFIG_BAR is not set`) and populate the overlay states.
2.  **Binary Impact Analysis**: The `ImpactAnalyzer` trait has a stub for `analyze_binary_impact`. This future feature would map configuration symbols to actual `Makefiles` or `.o` objects to determine how a config change impacts final binary size or compilation targets.
3.  **Interactive TUI/GUI**: Consider wrapping the engine in a Terminal UI (like `ratatui`) or a web interface to allow users to visually toggle nodes and see the BFS blast radius in real-time.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4creators) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
