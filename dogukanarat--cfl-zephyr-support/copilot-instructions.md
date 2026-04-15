## cfl-zephyr-support

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This is **cfl_zephyr_support** - a C library template/scaffold project designed to be cloned and migrated to create new C libraries. The workflow is:

1. Clone this repository with a new library name
2. Run `./scripts/migration.sh <NewLibName>` to rename all files and references (provide PascalCase name)
3. Manually fix any TODOs (update descriptions, copyright, implement actual functionality)
4. Build, test, and develop the new library

**Critical**: When making changes to this scaffold, remember they will be used as templates for future libraries. Maintain consistency and clarity in all template files.

## Build Commands

### Basic Build
```bash
mkdir build && cd build
cmake ..
make
```

### Build with Tests
```bash
mkdir build && cd build
cmake -DBUILD_TESTS=ON ..
make
ctest                    # Run all tests
ctest --verbose          # Run with detailed output
./test_cfl_zephyr_support       # Run test executable directly
```

### Build Shared Library
```bash
cmake -DBUILD_SHARED_LIBS=ON ..
make
```

### Install
```bash
mkdir build && cd build
cmake ..
make
sudo make install        # Installs to /usr/local by default

# Custom install prefix
cmake -DCMAKE_INSTALL_PREFIX=/custom/path ..
make install
```

### Code Formatting
```bash
./scripts/format.sh                           # Format all C files
clang-format -i src/myfile.c                  # Format specific file
clang-format --dry-run --Werror src/file.c    # Check without modifying
```

### CI Scripts
```bash
./ci/debug.sh                                 # Clean debug build + run tests
./ci/release.sh                               # Release build (optimized)
./ci/release.sh --clean                       # Clean release build
./ci/install.sh                               # Install to /usr/local (needs sudo)
./ci/install.sh --user                        # Install to ~/.local
./ci/install.sh --prefix=/opt/mylib           # Install to custom location
```

## Migration Script Usage

The core feature of this scaffold is the intelligent migration script:

```bash
./scripts/migration.sh MyNewLib              # Rename to MyNewLib (use PascalCase)
./scripts/migration.sh MyNewLib --dry-run    # Preview changes without applying
```

**Input Format**: Provide the library name in **PascalCase** (e.g., `MyNewLib`, `JsonParser`, `HttpClient`).

**How it works:**
- Takes PascalCase input and generates four case variations automatically:
  - **snake_case**: `my_new_lib` (used in filenames: `my_new_lib.c`, `my_new_lib.h`)
  - **lowercase**: `mynewlib` (used in directories: `include/mynewlib/`, packages)
  - **PascalCase**: `MyNewLib` (used in CMake project names, documentation)
  - **UPPERCASE**: `MY_NEW_LIB` (used in header guards, macro definitions)
- Intelligently applies the correct case in different contexts:
  - **Filenames**: snake_case → `my_new_lib.c`, `my_new_lib.h`, `my_new_lib_types.h`, `my_new_lib_int.c`
  - **Directories**: lowercase → `include/mynewlib/`
  - **CMake project**: PascalCase → `project(MyNewLib)`
  - **Header guards**: UPPERCASE → `#ifndef INC_MY_NEW_LIB_H`
  - **CMake configs**: lowercase → `mynewlibConfig.cmake.in`, `mynewlibTargets.cmake`
  - **Test files**: snake_case → `test_my_new_lib.c`
- Updates file contents in CMakeLists.txt, source/header files, README, CMake configs, test files
- Renames all files and directories throughout the project
- If input contains underscores (e.g., `My_New_Lib`), they are preserved in all variations

**Important**: If you add new template files, update the `files_to_update` array in `scripts/migration.sh` (around line 176-185) to include them.

## Code Architecture

### Template File Structure

All source and header files follow a strict organizational template with labeled sections:

**Headers (.h):**
```c
/* filename.h - one line definition */
/* All Rights Reserved */

#ifndef INC_FILENAME_H
#define INC_FILENAME_H

#ifdef __cplusplus
extern "C" {
#endif

/* Includes */
/* Configurations */
/* Definitions */
/* Types */
/* External Declarations */

#ifdef __cplusplus
}
#endif
#endif
```

**Source Files (.c):**
```c
/* filename.c - one line definition */
/* All Rights Reserved */

/* Includes */
/* Imports */
/* Definitions */
/* Types */
/* Forward Declarations */
/* Variables */
/* Functions */
```

**When editing template files**: Preserve these section markers. They provide consistent organization across all files.

### Directory Organization

- **include/cfl_zephyr_support/** - Public API headers (what users #include)
  - `cfl_zephyr_support.h` - Main API header
  - `cfl_zephyr_support_types.h` - Common types/definitions

- **src/** - Implementation files (not exposed to users)
  - `cfl_zephyr_support.c` - Core implementation
  - `cfl_zephyr_support_int.c` - Internal utilities (for helpers not in public API)

- **test/** - Unity test framework integration
  - Auto-downloads Unity v2.6.0 via CMake FetchContent
  - Test naming: `test_<function>_should_<behavior>_when_<condition>`

- **example/** - Example applications demonstrating library usage
  - `example.c` - Command-line application showcasing all library functions
  - Uses getopt for argument parsing and POSIX system calls
  - Demonstrates signal handling, error handling, and verbose logging
  - Uses only standard POSIX libraries (stdio, stdlib, string, unistd)
  - Easy to modify and adapt for different use cases
  - See `example/README.md` for customization guide

- **cmake/** - CMake modules
  - `cfl_zephyr_supportConfig.cmake.in` - Package config template (enables find_package())
  - `FindUnity.cmake` - Unity framework finder

- **scripts/** - Utility scripts
  - `migration.sh` - The core renaming script
  - `format.sh` - Format all C files with clang-format

- **docs/** - Documentation
  - `FORMATTING.md` - Code style guide

## Code Formatting Standards

Enforced via `.clang-format` and `.editorconfig`:

**Naming Conventions:**
- Functions/variables: `camelCase` (e.g., `myFunction`, `localVariable`)
- Defines/macros: `UPPER_CASE_WITH_UNDERSCORE` (e.g., `MAX_BUFFER_SIZE`)
- Global variables: `PascalCase` (e.g., `GlobalCounter`)

**Style Rules:**
- Braces: Allman style (braces on new lines)
- Pointers: Right-aligned, close to name (`int *ptr`, not `int* ptr`)
- Indentation: 4 spaces, no tabs
- Column limit: 100 characters
- Function parameters: One per line for long signatures, single line for short ones

**Example:**
```c
void shortFunction(int param)
{
    int *localVar = NULL;
}

status_t longFunction(
    int32_t handleId,
    const credentials_t *credential,
    state_t *state,
    uint32_t timeout)
{
    if (credential == NULL)
    {
        return STATUS_ERROR;
    }
    return STATUS_SUCCESS;
}
```

## CMake Build System Architecture

**Key features:**
- C99 standard enforced (public requirement)
- Compiler warnings: `-Wall -Wextra -Wpedantic` (GCC/Clang), `/W4` (MSVC)
- Symbol visibility control for shared libraries
- GNUInstallDirs for standard installation paths
- Component-based installation (Runtime/Development)
- CMake package config generation (enables `find_package(cfl_zephyr_support)`)
- Version file generation for SameMajorVersion compatibility
- Build summary output showing configuration
- Automatic default to Release build if not specified

**Build options:**
- `BUILD_SHARED_LIBS` - Build shared library instead of static (default: OFF)
- `BUILD_TESTS` - Build unit tests with Unity (default: OFF)
- `BUILD_EXAMPLES` - Build example applications (default: OFF)

**Installation outputs:**
- Libraries → `/usr/local/lib/`
- Headers → `/usr/local/include/cfl_zephyr_support/`
- CMake config → `/usr/local/lib/cmake/cfl_zephyr_support/`

**When adding new source files**: Update `target_sources()` in CMakeLists.txt (around line 57-62).

## Testing Architecture

Uses Unity Test Framework (ThrowTheSwitch/Unity) with intelligent fallback mechanism:

**Unity dependency resolution (automatic):**
1. First tries to find Unity installed locally via:
   - pkg-config (Linux)
   - CMake find_package
   - Common system paths (/usr/include, /usr/local/include, etc.)
2. If not found locally, downloads via FetchContent (v2.6.0) from GitHub
3. Option to force FetchContent: `-DFORCE_FETCH_UNITY=ON`

**Test properties:**
- Test name: `cfl_zephyr_support_unit_tests` (CTest)
- Timeout: 30 seconds per test
- Labels: "unit", "cfl_zephyr_support"
- Test files follow structure: setUp() → tests → tearDown() → main()

**Adding new test files:**
1. Create `test/test_mymodule.c`
2. Update `test/CMakeLists.txt`:
   - Add to `target_sources(test_cfl_zephyr_support PRIVATE test_mymodule.c)`
   - Or create a new test executable and register with `add_test()`

See `test/README.md` for comprehensive Unity assertions reference.

## CI Scripts Architecture

The `ci/` directory contains automation scripts for common development and deployment tasks:

### debug.sh - Debug Build & Test
**Purpose**: Clean debug build with full test execution
**Process**:
1. Removes build directory completely
2. Configures with `-DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTS=ON`
3. Builds with debug symbols and no optimization
4. Runs full test suite with `ctest --verbose --output-on-failure`

**Use**: Local development, CI/CD test pipelines, verifying changes

### release.sh - Release Build
**Purpose**: Optimized production build
**Process**:
1. Optionally cleans build directory (`--clean` flag)
2. Configures with `-DCMAKE_BUILD_TYPE=Release`
3. Builds with `-O3` optimization, no debug symbols
4. Creates optimized library

**Use**: Production builds, performance testing, releases

### install.sh - Installation
**Purpose**: Build and install library to system or custom location
**Features**:
- Auto-detects need for sudo (system paths vs user paths)
- Supports custom install prefix
- User-local install option (`--user` → `~/.local`)
- Configurable build type (Release/Debug)
- Post-install usage instructions

**Use**: System installation, user installation, CI/CD deployment

All scripts include:
- Error handling with immediate exit on failure
- Colored output (info/success/warning/error)
- Progress tracking with step numbers
- Summary reports
- Automatic parallel builds using `$(nproc)`

## GitHub Actions Workflows

The `.github/workflows/` directory contains automated CI/CD workflows:

### ci.yml - Main CI Workflow
**Purpose**: Comprehensive testing with matrix strategy

**Matrix Configuration**:
- Compilers: GCC and Clang
- Library types: Static and Shared
- Total: 4 configurations tested in parallel

**Triggers**: push (master/main/develop), pull_request (master/main), workflow_dispatch

**Process**:
1. Installs dependencies (CMake, Clang, clang-format, clang-tidy)
2. Caches CMake build directory (keyed by OS, compiler, library type, CMakeLists.txt)
3. Configures CMake with matrix-specific options
4. Builds with parallel jobs
5. Runs CTest with verbose output
6. Uploads test results and build artifacts

**Artifacts**: Test results and compiled libraries for each matrix configuration

### format-check.yml - Code Formatting Validation
**Purpose**: Enforce clang-format compliance

**Triggers**: push, pull_request, workflow_dispatch

**Process**:
1. Finds all .c and .h files in src/, include/, test/
2. Runs clang-format --dry-run --Werror
3. Fails if any formatting issues found

**Action Required**: Run `./scripts/format.sh` to fix formatting issues

### static-analysis.yml - Static Code Analysis
**Purpose**: Run clang-tidy static analysis

**Triggers**: push, pull_request, workflow_dispatch

**Process**:
1. Configures CMake to generate compile_commands.json
2. Runs clang-tidy with --warnings-as-errors='*' on all source files
3. Uses .clang-tidy configuration (13 check categories enabled)

**Benefits**: Catches bugs, performance issues, readability problems early

## What This Scaffold Provides

Template users receive:
- Complete CMake build system with installation support
- Unity test framework integration
- Example CLI application demonstrating library usage (POSIX-only, easily customizable)
- Code formatting enforcement (.clang-format, .editorconfig, .clang-tidy)
- GitHub Actions CI/CD workflows (build matrix, format check, static analysis)
- CI automation scripts (debug, release, install)
- Organized file templates with clear section markers
- Smart migration script preserving case conventions
- MIT License template
- Git ignore patterns for C/CMake projects
- Documentation structure

## What Template Users Must Do

After running migration.sh:
1. Review all renamed files
2. Update one-line descriptions in each file header
3. Update copyright from "All Rights Reserved" to their license
4. Update CMakeLists.txt project description
5. Implement actual library functionality
6. Add tests for their implementation
7. Update example applications to match their actual API
8. Update README with their library's purpose

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dogukanarat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dogukanarat)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
