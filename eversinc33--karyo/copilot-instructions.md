## karyo

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Karyo**

Karyo is a lightweight C++ library for symbolic and concrete execution of Remill-generated LLVM bitcode (x86). It lets you execute a Remill IR function with a mix of concrete and symbolic input register values, observe intermediate state via callbacks, and query output register values as concrete values or symbolic expression trees. Built specifically for VMP handler analysis and similar Remill-based reverse engineering workflows.

**Core Value:** Given a Remill handler in LLVM IR, determine the value of a register (e.g. EBP) at the dispatch point — as a concrete value or a symbolic expression over the handler's inputs.

### Constraints

- **Tech stack:** C++17 or C++20, CMake, links against LLVM (same version as lifter)
- **Dependencies:** Whatever is needed — correctness and usability over dep minimalism
- **Distribution:** Git submodule consumed by the lifter and potentially other projects
- **Performance:** Fast enough for iterating over hundreds of handlers without being a bottleneck; not latency-critical
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Dependency: LLVM
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| LLVM | **Match the lifter exactly** (likely 17 or 18) | IR traversal, type system, APInt arithmetic | Must be ABI-identical to the consuming project. Different versions cause ODR violations on `llvm::Value*`, `llvm::LLVMContext`, `llvm::Type*` pointers passed across the boundary |
#### LLVM APIs to Use
#include "llvm/IR/Module.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Support/raw_ostream.h"
#### LLVM APIs to Avoid
| API | Reason |
|-----|--------|
| `llvm::ExecutionEngine` / `llvm::Interpreter` | `GenericValue` union has no symbolic extension point; cannot add symbolic values without forking LLVM |
| `llvm::MCJIT` / `llvm::OrcJIT` | Precludes instruction-level callbacks; wrong abstraction |
| `llvm::PassManager` | Transforms IR; Karyo evaluates it |
| `llvm::IRBuilder` | Emits new IR; Karyo only reads IR |
| `getPointerElementType()` | Removed in LLVM 17 — use opaque pointer APIs |
### C++ Standard: C++17
| Standard | Assessment |
|----------|-----------|
| **C++17** | **Use this.** LLVM 17/18 and Remill both target C++17. GCC 9+, Clang 10+ have full support. Provides `std::variant`, `std::optional`, `std::string_view`, `if constexpr`, structured bindings — all needed. |
| C++20 | Skip — some distribution compilers have incomplete support; concepts add nothing functional here. |
| C++23 | Do not use — zero distribution support at LLVM-consuming project sites. |
- `std::variant<llvm::APInt, ExprRef>` — the Value discriminated union, no virtual dispatch
- `std::optional<Value>` — uninitialized register slots
- `std::function<bool(ExecutionState&)>` — stop predicate type
- `std::unordered_map<llvm::Value*, Value>` — interpreter SSA frame
- `if constexpr` — variant visitor dispatch
### Expression Tree / Symbolic Value Representation
### CMake Integration (Submodule Pattern)
# Karyo/CMakeLists.txt
# Do NOT hardcode LLVM version or path.
# Consumer sets LLVM_DIR before add_subdirectory(karyo); we reuse it.
# Link only what's used — never link all LLVM libs
# Option A: git submodule
- `include_directories()` / `link_libraries()` global — pollutes all targets
- `llvm_map_components_to_libnames(... all)` — links 300+ LLVM libs
- Hardcoding LLVM path in Karyo's CMakeLists — breaks CI
- Setting `CMAKE_BUILD_TYPE` inside library CMakeLists — never override consumer's build type
- Re-exporting LLVM include dirs as `PUBLIC` — consumers get LLVM from their own `find_package`
### Remill State Struct Integration
### Prior Art Lessons
| Tool | Key Lesson for Karyo | Do Not Copy |
|------|---------------------|------------|
| **LLVM Interpreter** (`lib/ExecutionEngine/Interpreter/`) | `InstVisitor` dispatch pattern is correct | `GenericValue` has no symbolic extension point |
| **haybale** (Rust) | `FunctionHooks` table (name → closure) for intrinsic interception; explicit `State` struct | Mandatory Boolector; all values symbolic (no concrete fast path) |
| **SymCC** (C++) | Concrete-first + symbolic shadow confirms `Value = variant<APInt, ExprRef>` | Requires recompiling target; too heavy |
| **Triton** (C++/Python) | Best callback/event API; eager constant folding in concrete domain | x86 instruction-level, not LLVM IR |
| **KLEE** (C++) | `ExecutionState` with `constraints` vector + `addressSpace` is the right path-state model | Mandatory Z3/STP; path scheduler overkill |
## Complete Dependency Summary
### Required
| Dependency | Obtained Via | Notes |
|------------|-------------|-------|
| LLVM (same version as lifter) | `find_package(LLVM CONFIG)` via consumer's `LLVM_DIR` | Version matched automatically |
| C++17 compiler | GCC ≥ 9, Clang ≥ 10 | Standard on Ubuntu 20.04+, RHEL 8+ |
| CMake ≥ 3.20 | System | FetchContent, generator expressions |
### Optional (Tests Only)
| Dependency | Obtained Via | Notes |
|------------|-------------|-------|
| GoogleTest | `FetchContent_Declare` in `tests/CMakeLists.txt` | Only when `KARYO_BUILD_TESTS=ON` |
| Remill headers | Consumer's build environment | Only for `static_assert` offset verification |
### Explicitly Excluded
| Dependency | Reason |
|------------|--------|
| Z3 / Bitwuzla / Boolector | SMT solver — explicit project out-of-scope |
| Boost | Heavy; C++17 stdlib covers all needs |
| gflags / glog | Remill internals; must not leak into Karyo consumers |
| LLVM ExecutionEngine | Wrong abstraction layer |
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [eversinc33/Karyo](https://github.com/eversinc33/Karyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
