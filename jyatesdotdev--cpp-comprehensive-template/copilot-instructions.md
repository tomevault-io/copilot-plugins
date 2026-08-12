## cpp-comprehensive-template

> This is the **navigation index** for coding agents. Every source directory has its own

# AGENTS.md — C++ Comprehensive Template (root index)

This is the **navigation index** for coding agents. Every source directory has its own
`AGENTS.md` with local rules and business context. **Read the one nearest to the file you
are editing** — it is the source of truth for that directory. This root file holds the
project-wide conventions that apply *everywhere*.

Modern **C++20**, CMake ≥ 3.21, dependencies via **vcpkg**. Each module = one namespace =
one CMake target = one `include/<module>/` dir (and optionally one `src/<module>/` dir).

---

## Directory map → where to look

| Directory | AGENTS.md | What lives there |
|---|---|---|
| `include/` | [include/AGENTS.md](include/AGENTS.md) | All public headers (the API surface) |
| `include/core/` | [core](include/core/AGENTS.md) | App lifecycle, RAII, PImpl (**STATIC lib**) |
| `include/memory/` | [memory](include/memory/AGENTS.md) | RAII handles, arena allocator, smart pointers (**STATIC lib**) |
| `include/concurrency/` | [concurrency](include/concurrency/AGENTS.md) | Thread pool, SPSC lock-free queue, parallel_for |
| `include/hpc/` | [hpc](include/hpc/AGENTS.md) | SIMD intrinsics, parallel-STL wrappers |
| `include/etl/` | [etl](include/etl/AGENTS.md) | Lazy pipeline, parallel MapReduce |
| `include/api/` | [api](include/api/AGENTS.md) | REST server/client (cpp-httplib + JSON) |
| `include/database/` | [database](include/database/AGENTS.md) | SQLite RAII wrapper, Repository ORM |
| `include/patterns/` | [patterns](include/patterns/AGENTS.md) | CRTP, type erasure, visitor, observer |
| `include/rendering/` | [rendering](include/rendering/AGENTS.md) | OpenGL RAII, shaders, render pipeline (gated) |
| `include/simulation/` | [simulation](include/simulation/AGENTS.md) | ECS, physics (Verlet), numerical (RK4) |
| `include/cli/` | [cli](include/cli/AGENTS.md) | CLI11 helpers, terminal output formatting |
| `src/` | [src/AGENTS.md](src/AGENTS.md) | Implementation `.cpp` files (only `core` + `memory`) |
| `tests/` | [tests/AGENTS.md](tests/AGENTS.md) | Catch2 unit tests (one file per module) |
| `examples/` | [examples/AGENTS.md](examples/AGENTS.md) | Runnable demos (one per module) |
| `benchmarks/` | [benchmarks/AGENTS.md](benchmarks/AGENTS.md) | Google Benchmark micro-benchmarks |
| `cmake/` | [cmake/AGENTS.md](cmake/AGENTS.md) | Custom CMake modules (security tooling) |
| `docs/` | [docs/AGENTS.md](docs/AGENTS.md) | Guides and reference documentation |
| `.github/workflows/` | [ci](.github/workflows/AGENTS.md) | CI: build/test, coverage, security scanning |

**Do not edit** `vcpkg_installed/` or `build/` — generated/vendored, never hand-modified.

---

## Project-wide conventions (apply in EVERY directory)

### Language & style
- **C++20**. `CMAKE_CXX_EXTENSIONS OFF` (no GNU extensions — standard C++ only).
- Headers use `#pragma once` (never include guards).
- Formatting is enforced by `.clang-format` (LLVM base, **4-space indent, 100-col limit**,
  attached braces, regrouped includes). Run `clang-format -i` before finishing.
- **Naming** (`.clang-tidy` + de-facto codebase style):
  - Types / classes / structs: `PascalCase` (`App`, `ThreadPool`, `RestServer`).
  - Namespaces: `lower_case` (`core`, `hpc`, `cli::fmt`).
  - Private data members: trailing underscore `_` (`impl_`, `handle_`, `db_`).
  - Functions & methods: **`lower_snake_case`** (STL-style: `make_widget`, `add_particle`,
    `col_int`) — `.clang-tidy` sets `FunctionCase: lower_case` to match. When in doubt,
    **match the surrounding file**.
  - Compile-time feature flags / macros: `UPPER_CASE` (`HAS_JSON`, `ENABLE_GL`).
- Every public entity gets a **Doxygen** comment (`///`, `@brief`, `@param`, `@tparam`,
  `@return`, `@throws`). This is a hard convention — see any existing header.

### Safety & idioms (this repo is security-focused)
- **RAII everywhere.** Prefer the *Rule of Zero*. If you own a raw resource, apply the
  *Rule of Five*: `delete` copy, provide `noexcept` move, release in the destructor.
- Never `new`/`delete` directly — use `std::make_unique`/`make_shared` or the RAII wrappers.
- Mark move ctor/assignment, swaps, destructors, and trivial getters `noexcept`.
- Use `[[nodiscard]]` on getters and factory returns.
- Prefer `std::string_view` / `std::span` for read-only params; pass sinks by value + move.
- The build is **warning-clean under strict flags** (`-Wall -Wextra -Wpedantic -Wshadow
  -Wconversion -Wsign-conversion -Wold-style-cast -Wnull-dereference …`). Fix warnings; do
  not suppress. Watch narrowing/sign conversions especially (cast explicitly & intentionally).
- **clang-tidy `cert-*`, `clang-analyzer-security.*`, and `concurrency-*` are errors.**
  CI fails on any such finding. See [docs/SECURITY_SCANNING.md](docs/SECURITY_SCANNING.md).

### Build & verify (run before declaring done)
```bash
cmake --preset default && cmake --build build/default   # configure + build
ctest --preset default                                  # run tests
```
Requires `VCPKG_ROOT` set. Useful presets: `debug`, `release`, `asan`, `ubsan`, `tsan`,
`msan`, `asan-ubsan`, `security-scan`, `coverage`. Sanitizers are **mutually exclusive**
where noted (ASan/TSan/MSan cannot combine). CI enforces **≥80% line coverage**.

---

## The golden rule for adding a feature to a module

A change to a module almost always touches **four** places — keep them in sync:
1. `include/<module>/…` — the header (declare + Doxygen-document the API).
2. `src/<module>/…` — the `.cpp`, **only if** the module is a STATIC lib (`core`, `memory`).
   All other modules are header-only (`INTERFACE`) — logic lives in the header.
3. `tests/<module>_tests.cpp` — a Catch2 test proving it (coverage gate is 80%).
4. `examples/<module>_demo.cpp` and/or `docs/` — keep demos and docs truthful.

To add a **whole new module**, also register a target in the root `CMakeLists.txt`, add a
test line in `tests/CMakeLists.txt`, and an example in `examples/CMakeLists.txt`. See
[docs/EXTENDING.md](docs/EXTENDING.md).

---

## Module quick-reference (target type & external deps)

| Module | Target type | Requires | Optional (compile-def) |
|---|---|---|---|
| core | STATIC | Threads | fmt `HAS_FMT`, spdlog `HAS_SPDLOG`, json `HAS_JSON`, boost `HAS_BOOST` |
| memory | STATIC | — | — |
| concurrency | INTERFACE | Threads | — |
| hpc | INTERFACE | Threads | `<execution>` → parallel STL |
| etl | INTERFACE | Threads | — |
| api | INTERFACE | **cpp-httplib** (target skipped if absent) | json `HAS_JSON` |
| database | INTERFACE | **SQLite3** (target skipped if absent) | — |
| patterns | INTERFACE | — | — |
| rendering | INTERFACE | **`ENABLE_RENDERING` + OpenGL** (entirely `#ifdef ENABLE_GL`) | glfw, glm |
| simulation | INTERFACE | — | — |
| cli | INTERFACE | CLI11 (FetchContent) | boost `HAS_BOOST` |

"Requires" targets are **conditionally created** — code guarded by `#ifdef HAS_JSON` /
`#ifdef ENABLE_GL` etc. must keep compiling when the dependency is absent.

---
> Source: [jyatesdotdev/cpp-comprehensive-template](https://github.com/jyatesdotdev/cpp-comprehensive-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
