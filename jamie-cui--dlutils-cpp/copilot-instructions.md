## dlutils-cpp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a header-only C++ library for dynamic loading utilities (dlutils-cpp). It provides a safe and convenient wrapper around dlopen/dlsym for dynamically loading shared libraries at runtime. The main use case demonstrated is loading OpenSSL's libcrypto.so.

## Code Architecture

1. **Core Library** (`include/dlutils/dlutils.hpp`):
   - `DlFun`: Template class wrapping function pointers from dynamically loaded libraries
   - `DlLibBase`: Base class for dynamic library loading with dlopen/dlsym wrapper functions
   - Utility functions for string creation and template argument deduction

2. **Example Usage** (`test/openssl.hpp`):
   - Concrete implementation showing how to use the library to load OpenSSL functions
   - `LibCrypto` class inheriting from `DlLibBase`
   - Example of loading `EVP_MD_CTX_new` function from libcrypto.so

3. **Build System**:
   - CMake-based build system with C++17 standard
   - GoogleTest integration for testing via ExternalProject
   - Header-only library structure

## Common Development Tasks

### Building
```bash
mkdir build && cd build
cmake ..
make
```

### Running Tests
```bash
mkdir build && cd build
cmake ..
make
ctest
# Or run specific test
./openssl_test
```

### Generating Coverage Reports
```bash
mkdir build && cd build
cmake -DENABLE_COVERAGE=ON ..
make
ctest  # Run tests to generate coverage data
make coverage  # Generate coverage report (lcov/html if available, gcov files otherwise)
# If lcov is installed: Open coverage_report/index.html in your browser to view the report
# If lcov is not installed: Gcov files will be generated in the gcov_report directory
```

### Adding New Functions
1. Add a new `DlFun<>` member to your library class (e.g., `LibCrypto`)
2. Add the function name to the `LoadAll()` method using `DLUTILS_SELF_DLSYM(NAME)`
3. The template arguments for `DlFun` should match the function signature

### Adding New Libraries
1. Create a new class inheriting from `DlLibBase`
2. Define `DlFun` members for each function you want to load
3. Implement `LoadAll()` method to load all functions
4. Set the correct library name in `LIB_NAME`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jamie-Cui)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jamie-Cui)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
