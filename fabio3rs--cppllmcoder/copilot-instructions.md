## cppllmcoder

> **cppllmcoder** is a local-first, high-performance recursive coding and reverse-engineering agent built for large codebases, firmware images, and long-horizon technical analysis. It features a C++23 orchestration engine, embedded Lua tool-calling, and a SQLite-based persistent memory ("The Brain").

# GEMINI.md - Project Context: cppllmcoder

## Project Overview
**cppllmcoder** is a local-first, high-performance recursive coding and reverse-engineering agent built for large codebases, firmware images, and long-horizon technical analysis. It features a C++23 orchestration engine, embedded Lua tool-calling, and a SQLite-based persistent memory ("The Brain").

### Core Technologies
- **Language:** C++23 (standard required).
- **Tool-Calling:** Lua 5.4 with [Sol3](https://github.com/ThePhD/sol2) for C++ interop.
- **Persistence:** SQLite 3 for task management, context compression (Pointers), and knowledge graphs.
- **Vector Search:** `sqlite-vec` or `sqlite-vss` for local semantic retrieval.
- **LLM Backends:** Ollama or llama.cpp (local-first), with optional cloud delegation.
- **Testing:** GoogleTest (GTest).

### Key Architectural Concepts
1.  **Fast Core, Minimal Scaffold:** The C++ engine manages execution, streaming, and persistence, while the LLM remains the primary planner.
2.  **Long-Context by Indirection:** Old findings are compressed into "Pointers" (IDs like `P_42`) with micro-summaries stored in SQLite to keep the active prompt small.
3.  **Recursive Task Decomposition (RLM):** Complex tasks spawn focused sub-agents in isolated Lua VMs.
4.  **Persistent Brain:** `brain.db` (located in `.cppllmcoder/`) ensures state continuity across sessions and crashes.
5.  **Programmable Tool-Calling:** The model emits Lua code within `<code>` tags instead of verbose JSON.
6.  **Time-Stamped Observability:** All core tables keep millisecond `created_at`/`updated_at` via SQLite defaults and triggers; `execution_logs` add `duration_ms`, and the TUI prefixes logs with `[HH:MM:SS.mmm]` to highlight latency hotspots.

---

## Directory Structure
- `src/`: C++23 runtime implementation. `main.cpp` is the CLI entry point.
- `include/`: Shared headers. `stdafx.hpp` is the precompiled-header (PCH) anchor.
- `tests/`: GoogleTest suites.
- `docs/`: Design documents, architectural blueprints, and research ideas (`IDEAS.md`, `REVERSE_ENGINEERING_IDEAS.md`).
- `.cppllmcoder/`: (Runtime) Stores `brain.db` and other persistent data.

---

## Building and Running

### Prerequisites
- C++23 compatible compiler (GCC 13+, Clang 16+, or MSVC 19.34+).
- CMake 3.12+.
- SQLite3.
- GoogleTest (optional, for tests).
- Lua 5.4 and Sol3 (planned dependencies).

### Commands
- **Configure:**
  ```bash
  cmake -S . -B build -DENABLE_TESTS=ON -DENABLE_SANITIZERS=ON
  ```
- **Build:**
  ```bash
  cmake --build build
  ```
- **Test:**
  ```bash
  ctest --test-dir build
  ```
- **Run:**
  ```bash
  ./build/cppllmcoder
  ```

---

## Development Conventions

### Coding Style
- **Standard:** C++23.
- **Indentation:** 4 spaces, no tabs.
- **Headers:** Keep lightweight; include `stdafx.hpp` to leverage PCH.
- **Error Handling:** Prefer `std::expected` and `std::optional`.
- **Performance:** Use `std::span` for zero-copy views and coroutines (`std::generator`) for streaming.
- **Warnings:** Treated as errors (`-Werror`).

### Naming Conventions
- **Namespaces:** `lower_snake_case`
- **Classes/Structs:** `PascalCase`
- **Functions/Methods:** `camelCase`
- **Variables:** `lower_snake_case`
- **Constants:** `kPascalCase`

### SQLite Schema (`brain.db`)
- `tasks`: Recursive tree of tasks/subtasks.
- `pointers`: Compressed micro-summaries and file references.
- `vector_index`: Virtual table for embeddings (`sqlite-vec`).
- `knowledge_graph`: Relations between pointers (e.g., `CALLS_FUNCTION`).
- `execution_logs`: Audit trail of Lua snippets and outputs, including `duration_ms` and automatic millisecond `timestamp`.

All core tables carry `created_at` / `updated_at` with millisecond precision via `DEFAULT (STRFTIME('%Y-%m-%d %H:%M:%f', 'NOW'))` plus `AFTER UPDATE` triggers, so the C++ runtime does not have to manage timestamps manually.

---

## Current Status & Roadmap
The project is currently in the **architectural foundation** phase. The build system and basic "Hello World" are established.
1. **Milestone 1:** `brain.db` bootstrap (Schema & CRUD).
2. **Milestone 2:** Lua embedding with Sol3 and `<code>` execution.
3. **Milestone 3:** Pointer-based context compression loop.
4. **Milestone 4:** Recursive sub-agent spawning (`rlm.spawn()`).
5. **Milestone 5:** Semantic retrieval integration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fabio3rs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Fabio3rs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
