## cmake-start

> **Always use modern C++ features:**

# Development Guidelines for AI Assistance

## Code Style & Language

**Always use modern C++ features:**
- Use C++23/26 features when available
- Prefer modules over traditional headers where supported
- Use concepts for template constraints
- Prefer `std::ranges` and `std::views` which both can be shortened to namespace `r::` and `v::`

**Always write clear, efficient code:**
- Use meaningful variable and function names
- Name variables and functions in snake_case
- Name classes, structs, and enums in CamelCase
- Prefix private member variables with `m_`
- Use UPPER_CASE for enum constants, static constexpr, and global constants
- Use lowercase for namespaces
- Prefer value semantics and move semantics
- Use `const` and `constexpr` where applicable

**Always follow these patterns:**
- Use RAII for resource management
- Prefer stack allocation over heap when possible
- Use smart pointers `unique_ptr` for ownership semantics
- Implement proper copy/move constructors and assignment operators

## Build System

**Always use the `llm-vcpkg` preset for AI-assisted development:**
- Configure: `cmake --preset llm-vcpkg`
- Build: `cmake --build --preset llm-vcpkg`
- This preset enables maximum safety checks (ASan, UBSan, clang-tidy, cppcheck)
- Warnings are treated as errors in this preset

**Always fix issues before committing:**
- Run `./fix_and_format.sh` to auto-fix clang-tidy issues and format code
- Address any remaining clang-tidy or cppcheck warnings manually
- Ensure the build passes with zero warnings

## Testing & Debugging

**Always write comprehensive tests:**
- Design for dependency injection
- Create unit tests for core functionality in the `tests/` directory
- Test edge cases and error conditions
- Run tests using `./run_test.sh` which outputs lcov coverage information

**Always aim for high code coverage:**
- Target at least 95% code coverage
- Review coverage reports to identify untested code paths

**Always provide clear error messages:**
- Include context in error messages
- Use `std::expected` or similar for error handling
- Log important state changes

## Project Structure

**Always place code in the correct directory:**
- `include/` - Public headers
- `src/` - Implementation files
- `tests/` - Unit tests (use Catch2 or similar)
- `bench/` - Benchmarks (use Google Benchmark)
- `playground/` - Experimental/prototype code
- `libs/` - Internal libraries
- `cmake/` - CMake modules and helpers

## Documentation

**Always include clear explanations:**
- Document complex algorithms and data structures
- Explain why design decisions were made
- Include usage examples for APIs

**Always provide build instructions:**
- Document required dependencies in vcpkg.json
- Explain CMake preset usage
- List supported platforms

## Problem-Solving Approach

**Always be technically precise:**
- Use correct terminology
- Provide concrete examples
- Reference relevant documentation or standards

**Always suggest cutting-edge solutions:**
- Leverage newest C++ features
- Recommend modern libraries and tools
- Propose performance optimizations

**Always verify code quality:**
- Ensure code compiles with the `llm-vcpkg` preset
- Run `fix_and_format.sh` before presenting solutions
- Verify test coverage meets the 95% threshold

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisi5700) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
