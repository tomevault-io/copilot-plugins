## loredb

> - Use PascalCase (UpperCamelCase) for class, struct, enum, and type names (e.g., `PageStore`, `ErrorCode`).


## Naming Conventions

- Use PascalCase (UpperCamelCase) for class, struct, enum, and type names (e.g., `PageStore`, `ErrorCode`).
- Use snake_case (lower_case_with_underscores) for functions, variables, and method names (e.g., `get_page_count`, `page_id`).
- Use UPPER_CASE for constants and macros (e.g., `PAGE_SIZE`, `INVALID_PAGE_ID`).
- Namespace names should be all lowercase and may be nested (e.g., `loredb::storage`).
- Do not use `using namespace std;` at the top level.

## Formatting

- Indent with 4 spaces. Do not use tabs.
- Place opening braces on the same line as the declaration.
- Limit lines to 100 characters where practical.
- Group standard library includes first, then project headers.
- Always use `#pragma once` in header files.

## Modern C++ Practices

- Use C++20 features (or newer if toolchain allows).
- Prefer smart pointers (`std::unique_ptr`, `std::shared_ptr`) and `std::span` over raw pointers.
- Use `std::optional`, `std::variant`, and `tl::expected` (aliased as `util::expected`) for error handling and optional values.
- Prefer `fmt` or `std::format` for string formatting; avoid C-style I/O.
- Use RAII for resource management. Implement the rule-of-five for classes managing resources.
- Avoid global variables and minimize use of singletons.

## Error Handling

- Use `util::expected<T, Error>` for error propagation; avoid error codes as return values.
- Define custom error types as needed (see `Error`, `ErrorCode`).
- Use exceptions only for truly exceptional, unrecoverable errors.

## Logging

- Use the provided `Logger` wrapper (backed by `spdlog`) for all logging.
- Use the provided macros (`LOG_INFO`, `LOG_ERROR`, etc.) for consistency.
- Prefer structured logging helpers for operations and performance events.

## Testing

- All new code must be covered by unit tests using GoogleTest.
- Place tests in the `tests/` directory, mirroring the module structure.
- Use descriptive test names and group related tests in test fixtures.

## Build & Tooling

- All code must build cleanly with CMake and vcpkg dependencies.
- Enable static analysis (clang-tidy, cppcheck) and treat warnings as errors in CI.
- Use `#pragma once` instead of include guards.
- Document all public APIs and classes with Doxygen-style comments.

## Documentation & Comments

- Prefer high-level documentation and code examples over excessive inline comments.
- Use Doxygen-style comments for all public APIs, classes, and complex functions.
- Add rationale for non-obvious design decisions in comments or documentation.

## Project-Specific Architecture

- All new modules should follow the established namespace and directory structure.
- New storage or query modules must implement the relevant interface and be registered appropriately.
- For database internals, prefer flat containers and minimize heap allocations in hot paths.
- Use concurrency primitives (`std::mutex`, `std::atomic`, etc.) and lock-free structures where appropriate.

## Cursor Rule Maintenance

- Keep rules atomic and specific; update as the codebase or team standards evolve.
- Version control this file and review changes as part of code review.
- Add comments to explain the rationale for complex or opinionated rules.

- For database internals, prefer flat containers and minimize heap allocations in hot paths.
- Use concurrency primitives (`std::mutex`, `std::atomic`, etc.) and lock-free structures where appropriate.

## Cursor Rule Maintenance

- Keep rules atomic and specific; update as the codebase or team standards evolve.
- Version control this file and review changes as part of code review.
- Add comments to explain the rationale for complex or opinionated rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buddy-computer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
