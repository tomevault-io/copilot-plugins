## ctopo

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ctopo**

C++20 header-only TDA (Topology Data Analysis) 库，以插件包形式面向商业/开源 CAx 内核 provider，赋予即插即用的 SOTA TDA 能力。采用 runtime 决策模型，博采众长——数据驱动算法选择，而非 compile-time 固化单一方案。

**Core Value:** 让 CAx 几何内核获得生产级 TDA 能力——形状识别、特征提取、网格分析——以 header-only 形式交付，零依赖，MIT 授权。

### Constraints

- **License**: MIT — 商业友好
- **ABI**: 长期稳定性保证，面向插件生态
- **C++ 标准**: C++20 mandatory
- **Header-only**: 零外部依赖，集成方无额外构建负担
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|----------------|
| CMake | 3.21+ | Build system | Industry standard for C++; FetchContent available since 3.14, header-only INTERFACE library pattern mature |
| C++20 | ISO/IEC 14882:2020 | Language | Concepts, ranges, coroutines, jthread with stop_token enable modern zero-overhead abstractions |
| std::pmr | C++17/20 | Memory allocation | Polymorphic_allocator and memory_resource provide runtime-flexible allocation without external dependencies |
| std::jthread + atomics | C++20 | Parallelism | Cooperative cancellation via stop_token, atomic wait/notify; no external library needed |
| std::ranges | C++20 | Algorithm API | Composable, lazy evaluation; ideal for TDA's filter-then-map-reduce patterns |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None (zero dependencies) | - | Philosophy | MIT header-only delivery; integration burden zero |
| std::unordered_map | C++11 | Default hash map | Single-threaded TDA; no external deps |
| robin-hood-hashing (optional) | latest | Memory optimization | Only if profiling shows hash collision overhead; via FetchContent only |
| Google SparseHash (optional) | latest | Memory optimization | Only for extreme memory constraints; via FetchContent only |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| CMake 3.21+ | Build/config | Use `FetchContent_MakeAvailable()` for dependencies |
| CMake INTERFACE library | Header-only packaging | `add_library(ctopo INTERFACE)` with generator expressions |
| CMake PRESET | Reproducible builds | `cmake --preset` for developers; `-DCMAKE_BUILD_TYPE=Release` |
| CMake Export/Install | Distribution | Generate Config.cmake for `find_package()` consumption |
## Installation
### Header-Only Integration (No Build)
# Consumer's CMakeLists.txt - zero build burden
### Development with Dependencies
# For optional dependencies (only if needed)
### Build and Install (For Plugin Distribution)
# In ctopo's CMakeLists.txt
## Alternatives Considered
| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| CMake INTERFACE | Bazel/Blaze | Only if integration ecosystem demands it; CMake has superior CAx kernel support |
| std::pmr | TBB (Threading Building Blocks) | TBB if complex parallel patterns (reductions, pipelines) needed; adds Intel dependency |
| std::jthread | std::async + future | std::async lacks cancellation control; jthread's stop_token is superior for long-running TDA computations |
| std::unordered_map | absl::flat_hash_map | Google Abseil if already in ecosystem; adds dependency for marginal hash performance |
| No external memory pool | Boost.Pool | Boost adds significant weight; std::pmr::synchronized_pool_resource covers most cases |
## What NOT to Use
| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Boost | Massive dependency; contradicts header-only zero-burden goal | std::pmr with custom memory_resource |
| OpenMP | Compiler-specific pragmas; awkward with runtime algorithm selection | std::jthread + std::atomic |
| std::thread (raw) | No cooperative cancellation; requires manual join/detach | std::jthread with stop_token |
| MPI | Distributed computing; out of scope per PROJECT.md | std::jthread for单机 parallelism |
| GUDHI as dependency | Not header-only; complex build; overkill | Reference GUDHI algorithms, implement leaner version |
| Pre-C++20 allocators | Lacks std::pmr polymorphic flexibility | std::pmr::polymorphic_allocator |
## Stack Patterns by Variant
- Use only C++20 standard library
- CMake INTERFACE library with no FetchContent
- std::pmr::new_delete_resource for all allocations
- Single-threaded initial release; std::jthread for future parallelism
- Use std::pmr::synchronized_pool_resource for small object pooling
- Use std::pmr::monotonic_buffer_resource for filtration construction
- Implement custom memory_resource for TDA-specific allocation patterns
- Use std::jthread with stop_token for cancellable parallel loops
- Use std::atomic for lock-free reduced boundary matrix operations
- Use std::latch/barrier for phase synchronization
- Defer TBB evaluation until profiling proves std::jthread insufficient
## Version Compatibility
| Component | Compatible With | Notes |
|-----------|-----------------|-------|
| CMake | 3.14+ (FetchContent), 3.21+ (PRESET) | 3.14 minimum for FetchContent, 3.21 for modern presets |
| C++20 | GCC 10+, Clang 10+, MSVC 2019 16.6+ | Verify compiler support; some TDA code may need 11+ |
| std::pmr | C++17 | Widely supported; no concerns |
| std::jthread | C++20 | Requires C++20; ensure integration environment supports |
## Technical Debt Notes
## Sources
- [CMake FetchContent Documentation](https://cmake.org/cmake/help/latest/module/FetchContent.html) -- HIGH
- [CMake Importing/Exporting Guide](https://cmake.org/cmake/help/latest/guide/importing-exporting/index.html) -- HIGH
- [cppreference: C++ thread](https://en.cppreference.com/w/cpp/thread) -- HIGH
- [cppreference: C++ memory](https://en.cppreference.com/w/cpp/memory) -- HIGH
- [Ripser GitHub](https://github.com/Ripser/ripser) -- MEDIUM (verified C++11, header-only, Makefile)
- [GUDHI GitHub](https://github.com/GUDHI/gudhi-devel) -- MEDIUM (verified CMake, modular, not header-only)
- [DIPHA GitHub](https://github.com/DIPHA/dipha) -- MEDIUM (verified CMake, not header-only)
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [plasmayang/ctopo](https://github.com/plasmayang/ctopo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
