## template-metaprograming

> - `include/meta/`: Header-only utilities (C++23) used across the project. Keep utilities small and composable.

# Repository Guidelines

## Project Structure & Module Organization

- `include/meta/`: Header-only utilities (C++23) used across the project. Keep utilities small and composable.
- `src/`: Placeholder for compiled sources if needed later; current library is header-only.
- `tests/`: GoogleTest-based unit tests (`test_*.cpp`).
- `.github/workflows/ci.yml`: GitHub Actions workflow (build + test).

## Build, Test, and Development Commands

- Configure: `cmake -S . -B build`
- Build: `cmake --build build -j`
- Test: `ctest --test-dir build --output-on-failure`
- Warnings are treated as errors by default.

## Coding Style & Naming Conventions

- C++23, no extensions; warnings: `-Wall -Wextra -Wpedantic -Werror`.
- Indentation: 2 spaces; keep lines short and readable.
- Headers: snake_case (`include/meta/fixed_string.hpp`).
- Templates: prefer `using` aliases over `struct` when possible; avoid one-letter identifiers in public APIs.
- Keep new utilities header-only unless a strong reason exists.

## Testing Guidelines

- Framework: GoogleTest. Add or update tests in `tests/` mirroring the feature under `include/meta/`.
- Naming: `TEST(SuiteName, CaseName)`; file names `test_<area>.cpp` (e.g., `tests/test_typelist_advanced.cpp`).
- Scope: Cover success and failure at compile-time when feasible (use `static_assert` alongside runtime expectations).

## Commit & Pull Request Guidelines

- Commits: Clear, imperative subject (e.g., `Add unique_t and tests`). Keep focused; include rationale in the body when non-trivial.
- PRs: Describe motivation, key changes, and tests. Link issues when relevant. For meta utilities, include examples of intended usage.
- CI must pass. Avoid unrelated refactors in feature PRs.

## Architecture & Contribution Tips

- The `meta` library centers on type/value lists, detection, concepts, and constexpr algorithms. Prefer minimal dependencies and compile-time validation.
- When adding algorithms (e.g., `partition_t`, `zip`, `flatten`), preserve existing naming patterns and order stability.
- If you introduce runtime components under `src/`, wire them via CMake targets and add corresponding tests.

---
> Source: [Shagai/Template_Metaprograming](https://github.com/Shagai/Template_Metaprograming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
