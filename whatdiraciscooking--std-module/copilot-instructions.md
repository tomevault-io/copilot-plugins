## std-module

> This document provides comprehensive guidance for AI assistants working with the `std_module` repository.

# CLAUDE.md - AI Assistant Guide for std_module

This document provides comprehensive guidance for AI assistants working with the `std_module` repository.

## Project Overview

**std_module** is a C++20 module wrapper library for the C++ standard library. It enables developers to use modern `import` statements instead of traditional `#include` directives.

- **Language:** C++20 with modules
- **Version:** 0.1.0
- **Build System:** CMake 3.28+ with Ninja (required)
- **Status:** Production-ready - 72 standard library modules implemented with comprehensive tests

### Core Philosophy

The project prioritizes **flexibility over opinion**:
- No forced dependencies between modules
- Opt-in architecture - build only what you need
- Multiple integration methods (subdirectory, installed, manual)
- Standard library semantics preserved exactly (no modifications)

## Repository Structure

```
/home/user/std_module/
├── CMakeLists.txt              # Root build config
├── README.md                   # User-facing documentation (lean, pattern-focused)
├── CLAUDE.md                   # This file - comprehensive AI guide
├── src/                        # Module implementations (72 .cppm files)
│   ├── CMakeLists.txt         # Module build targets (uses macros)
│   ├── format.cppm            # Example: <format> wrapper
│   ├── vector.cppm            # Example: <vector> wrapper
│   ⋮
├── test/                       # Test suite (72 test files)
│   ├── CMakeLists.txt         # Test build config (uses macros)
│   ├── README.md              # Test suite documentation
│   ├── test_format.cpp        # Example: comprehensive format tests
│   ⋮
├── cmake/                      # CMake infrastructure
│   ├── StdModuleMacros.cmake  # Helper macros for adding modules
│   └── std_module-config.cmake.in
└── scripts/                    # Automation scripts
    ├── README.md
    └── symbol_coverage.py     # Symbol coverage analysis tool
```

### Implemented Modules (72 Total)

**Note:** README.md contains the canonical, exhaustive list of all 72 implemented modules in the "Available Modules" table. That table is the **single source of truth** for module availability, status, and special notes.

**Categories overview:**
- **Algorithms & Iterators:** algorithm, functional, iterator, ranges
- **Containers:** any, bitset, deque, forward_list, initializer_list, list, map, optional, queue, span, variant, vector
- **I/O:** fstream, ios, iosfwd, iostream, istream, iomanip (⚠️), syncstream
- **Text & Formatting:** charconv, format, locale, string_view
- **Concurrency:** barrier, condition_variable, coroutine, future (⚠️), latch, semaphore
- **Utilities:** bit, compare, concepts, exception, execution, filesystem, limits, memory_resource, new, numbers, numeric, random, source_location, system_error, typeindex

**Special cases:**
- `<new>` → `std_module.new_` (underscore to avoid keyword conflict)
- `<iomanip>` → ⚠️ Manipulators unusable due to ADL limitations
- `<future>` → ⚠️ `packaged_task` unusable, other components work

### README.md Documentation Philosophy

**IMPORTANT:** The README.md follows a **lean, pattern-focused approach**. This is a deliberate design choice to keep user-facing documentation concise and maintainable.

**Key principles:**

1. **Describe patterns, not exhaustive lists**
   - CMake options: Show the pattern `STD_MODULE_BUILD_<NAME>`, list the 3 special options, note defaults
   - Library targets: Show the pattern `std_module::<name>`, give 3-4 examples
   - **Don't** create tables listing all 72 modules in multiple places

2. **Single source of truth**
   - The "Available Modules" table is the **ONLY** place that exhaustively lists all 72 modules
   - Everything else refers to this table
   - When adding a module, update **only** this table in README.md

3. **Assume user knowledge**
   - **Don't** explain what `<vector>` does or what features `<algorithm>` provides
   - Users of this library already know the standard library
   - Focus on: module name, import statement, build options, known limitations

4. **Use vertical ellipses in file trees**
   - Example: `├── format.cppm`, `├── vector.cppm`, `⋮`
   - Saves space while conveying "many files here"

5. **Be concise everywhere except "Available Modules"**
   - Short paragraphs
   - Minimal examples
   - Only essential information

**What this means when updating README.md:**

- ✅ Add new module to "Available Modules" table with status and notes
- ✅ Keep examples to 3-4 items max (don't list all 72 modules)
- ✅ Use patterns instead of exhaustive enumerations
- ❌ Don't add the module to CMake Options table (removed - uses pattern now)
- ❌ Don't add the module to Library Targets table (removed - uses pattern now)
- ❌ Don't describe what the standard library component does
- ❌ Don't create new exhaustive lists

**Rationale:** With 72 modules (and growing toward 90+), exhaustive tables become:
- Hard to maintain (update in N places for each new module)
- Noisy for users (too much scrolling)
- Redundant (same information repeated multiple ways)

**This file (CLAUDE.md)** remains comprehensive and detailed - it's the AI assistant guide. README.md is for human users who want to get started quickly.

## Critical C++20 Module Concepts

### Module File Structure (.cppm)

All module files follow this pattern:

```cpp
module;                          // Global module fragment start
#include <header>                // Include standard library headers
export module std_module.name;   // Module declaration
export namespace std {           // Export namespace
    using std::symbol;           // Re-export symbols
}
```

**Example:** `src/format.cppm:1-47`

### Module Compilation Requirements

1. **Ninja generator is MANDATORY** - Make will not work
2. **CMake 3.28+** - Earlier versions lack proper module support
3. **Modern compiler:**
   - Clang 16+ (recommended)
   - GCC 11+ (experimental support)
   - MSVC 19.30+ (Visual Studio 2022)

### Module Naming Convention

- **Module name:** `std_module.{header}` (e.g., `std_module.format`)
- **CMake target:** `std_module::{header}` (e.g., `std_module::format`)
- **File name:** `{header}.cppm` (e.g., `format.cppm`)
- **Import statement:** `import std_module.{header};`

### Compiler and Standard Library Compatibility

**TESTED CONFIGURATIONS (2025-11-14):**

With operator exports (see section 8), pure module-only compilation works on all practical configurations:

| Compiler | Std Library | Status | Notes |
|----------|-------------|--------|-------|
| **Clang 18.1.3** | **libstdc++ 13** | ✅ **Works** | Most common Linux config |
| **Clang 18.1.3** | **libc++ 18** | ✅ **Works** | Common on macOS |
| **GCC 14.2.0** | **libstdc++ 14** | ✅ **Works** | Latest GCC |
| GCC 14.2.0 | libc++ | ❌ Not supported | GCC doesn't support `-stdlib=` flag |

**Key Findings:**

1. **C++20 standard** (set via `CMAKE_CXX_STANDARD 20`)
2. **Operator exports are CRITICAL** - Without them:
   - Clang + libstdc++: String output shows memory addresses
   - Clang + libc++: Build fails with concept redefinition errors
   - GCC + libstdc++: Build fails with various template errors
3. **Pure module-only works** - NO `#include` directives needed when operators are exported
4. **Mixing import + include causes issues** - Don't mix `import std_module.header;` with `#include <header>`

**Testing with different configurations:**

```bash
# Clang + libstdc++ (default)
cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++

# Clang + libc++
cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_CXX_FLAGS="-stdlib=libc++" \
  -DCMAKE_EXE_LINKER_FLAGS="-stdlib=libc++ -lc++abi"

# GCC + libstdc++ (default)
cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=g++-14
```

**Reference Implementation:**
- See `src/iostream.cppm` for operator export pattern
- See `test/test_iostream.cpp` for pure module-only test (no includes)

## Build System Architecture

### CMake Build Options

**Global Options:**

| Option | Default | Description |
|--------|---------|-------------|
| `STD_MODULE_BUILD_TESTS` | ON | Build test executables |
| `STD_MODULE_BUILD_ALL_MODULES` | ON | Build all available modules |
| `STD_MODULE_INSTALL` | ON | Generate installation targets |

**Per-Module Options:**

Each module has a dedicated option: `STD_MODULE_BUILD_<NAME>` (default: ON)

Examples:
- `STD_MODULE_BUILD_FORMAT=ON` - Build format module
- `STD_MODULE_BUILD_VECTOR=ON` - Build vector module
- `STD_MODULE_BUILD_ALGORITHM=ON` - Build algorithm module

See README.md "Available Modules" table for the complete list of all 72 modules (each has a corresponding `STD_MODULE_BUILD_<NAME>` option).

### Standard Build Workflow

```bash
# 1. Configure (Ninja is required)
cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++

# 2. Build all targets
cmake --build build

# 3. Run tests
ctest --test-dir build --output-on-failure

# 4. Optional: Install
cmake --install build --prefix /usr/local
```

### Module Target Pattern

**Modern Approach (Macro-Based):**

The project uses CMake macros to reduce boilerplate. Adding a new module is a one-liner:

```cmake
# In src/CMakeLists.txt
std_module_add_module(format)
std_module_add_module(vector)
```

This macro (defined in `cmake/StdModuleMacros.cmake`) automatically:
1. Creates library target `std_module_<name>`
2. Adds `<name>.cppm` using `FILE_SET CXX_MODULES`
3. Sets C++20 requirement
4. Creates namespaced alias `std_module::<name>`
5. Configures installation if enabled

**Traditional Approach (What the Macro Does):**

```cmake
# Manual pattern (now abstracted by std_module_add_module)
add_library(std_module_<name>)
target_sources(std_module_<name>
    PUBLIC FILE_SET CXX_MODULES FILES <name>.cppm)
target_compile_features(std_module_<name> PUBLIC cxx_std_20)
add_library(std_module::<name> ALIAS std_module_<name>)
install(TARGETS std_module_<name> EXPORT std_module-targets)
```

**Critical:** Use `FILE_SET CXX_MODULES` for module files, NOT regular `target_sources()`!

**Testing Macro:**

Similarly, adding tests is streamlined:

```cmake
# In test/CMakeLists.txt
std_module_add_test(format)
std_module_add_test(vector)
```

This creates the test executable, links it to the module, and registers with CTest.

## Infrastructure Patterns

### Overview of CMake Macro System

The project uses a sophisticated macro-based build system to minimize boilerplate and ensure consistency across all 72 modules. This is a **key pattern** to understand and follow.

**Location:** `cmake/StdModuleMacros.cmake`

**Three Core Macros:**

1. **`std_module_add_module(name)`** - Adds a module library
2. **`std_module_add_test(name)`** - Adds a test executable
3. **`std_module_add_to_aggregate(name)`** - Adds module to `std_module::all` target

### The "Three-Line Addition" Pattern

When adding a new standard library module, you touch **three CMake files** with **one line each**:

```cmake
# 1. src/CMakeLists.txt - Add the module
std_module_add_module(newmodule)

# 2. test/CMakeLists.txt - Add the test
std_module_add_test(newmodule)

# 3. src/CMakeLists.txt (aggregate section) - Add to "all" target
std_module_add_to_aggregate(newmodule)
```

**Why This Matters:**
- Consistency: All 72 modules follow identical patterns
- Maintainability: Changes to build logic happen in one place (the macro)
- Simplicity: Adding a module doesn't require understanding complex CMake

### File Naming Conventions

The project enforces strict naming conventions for automatic discovery:

| File Type | Pattern | Example |
|-----------|---------|---------|
| Module source | `src/{name}.cppm` | `src/format.cppm` |
| Test source | `test/test_{name}.cpp` | `test/test_format.cpp` |
| CMake target | `std_module_{name}` | `std_module_format` |
| CMake alias | `std_module::{name}` | `std_module::format` |
| CMake option | `STD_MODULE_BUILD_{NAME}` | `STD_MODULE_BUILD_FORMAT` |
| Module name | `std_module.{name}` | `std_module.format` |
| Import statement | `import std_module.{name};` | `import std_module.format;` |

**Special Case:** The `<new>` header becomes `new_.cppm` and `import std_module.new_;` because `new` is a C++ keyword.

### Symbol Coverage Analysis

**Location:** `scripts/symbol_coverage.py`

The project includes an automated symbol coverage analyzer that:
- Parses `.cppm` files to extract exported symbols
- Parses test files to find used symbols
- Reports which exported symbols are tested vs. untested
- Runs automatically as part of `ctest`

**Usage:**

```bash
# Manual run for specific module
python3 scripts/symbol_coverage.py format

# Check all modules
python3 scripts/symbol_coverage.py --all

# Runs automatically with tests
ctest --test-dir build
```

**Integration:** The coverage test is registered in `test/CMakeLists.txt` and runs alongside module tests.

### Aggregate Target Pattern

**Location:** `src/CMakeLists.txt` (bottom section)

The `std_module::all` target is an **INTERFACE library** that depends on all enabled modules:

```cmake
add_library(std_module_all INTERFACE)
add_library(std_module::all ALIAS std_module_all)

std_module_add_to_aggregate(format)
std_module_add_to_aggregate(vector)
# ... all 72 modules
```

**Usage by consumers:**

```cmake
# Link everything at once
target_link_libraries(myapp PRIVATE std_module::all)
```

### Option Checking Pattern

All macros check **two conditions** before building a module:

```cmake
if(STD_MODULE_BUILD_ALL_MODULES OR STD_MODULE_BUILD_${MODULE_NAME_UPPER})
    # Build the module
endif()
```

This enables both:
- Global control: `STD_MODULE_BUILD_ALL_MODULES=OFF` disables everything
- Granular control: `STD_MODULE_BUILD_FORMAT=ON` enables specific modules

**Common Build Patterns:**

```bash
# Build everything (default)
cmake -B build -G Ninja

# Build only format and vector
cmake -B build -G Ninja \
  -DSTD_MODULE_BUILD_ALL_MODULES=OFF \
  -DSTD_MODULE_BUILD_FORMAT=ON \
  -DSTD_MODULE_BUILD_VECTOR=ON

# Build everything except iomanip
cmake -B build -G Ninja \
  -DSTD_MODULE_BUILD_IOMANIP=OFF
```

### Testing Infrastructure Pattern

**Location:** `test/CMakeLists.txt`

The test infrastructure follows a consistent pattern:

```cmake
# 1. Enable testing
enable_testing()

# 2. Add module tests (one line per module)
std_module_add_test(format)
std_module_add_test(vector)
# ... etc.

# 3. Add symbol coverage as a test
add_test(NAME symbol-coverage
    COMMAND ${Python3_EXECUTABLE} ${CMAKE_SOURCE_DIR}/scripts/symbol_coverage.py --all
)
```

**Key Points:**
- Each module test is independent (can run in parallel)
- Tests are automatically registered with CTest
- Symbol coverage runs as part of the test suite
- All tests use the pattern: `import std_module.X;` then test functionality

## Development Workflows

### Adding a New Standard Library Module

Follow this checklist when wrapping a new stdlib header:

#### 1. Create Module Implementation File

Create `src/{header}.cppm` following the pattern in `src/format.cppm:1-47`:

```cpp
module;
#include <header>
export module std_module.header;

export namespace std {
    // Export all public APIs from the header
    using std::function_name;
    using std::class_name;
    // ... etc
}
```

**Location:** `src/{header}.cppm`

#### 2. Add Module to Build System (3 Lines)

Add one line to each of these files:

**File 1:** `src/CMakeLists.txt` (in the module list section ~line 18-42):
```cmake
std_module_add_module(header)
```

**File 2:** `src/CMakeLists.txt` (in the aggregate section ~line 60-84):
```cmake
std_module_add_to_aggregate(header)
```

**File 3:** `test/CMakeLists.txt` (in the test list section ~line 14-38):
```cmake
std_module_add_test(header)
```

**Note:** The CMake option `STD_MODULE_BUILD_HEADER` is auto-generated by the macro system.

#### 3. Create Test Following Testing Guidelines

Create `test/test_{header}.cpp` following the **Testing Philosophy and Guidelines** section below.

**Quick summary:**
- Choose **Tier 1** (50-100 lines) for simple modules or **Tier 2** (200-400 lines) for important modules
- Use the Tier 1 template as starting point (see Testing Guidelines section)
- Import the module: `import std_module.header;`
- Only include allowed headers: `<cassert>`, `<iostream>`, `<sstream>`
- Test basic usage, iterators, and ADL operators (with try/catch)
- Use assertions and checkmarks (✓) for validation
- Document FUTURE tests that need unavailable modules
- See detailed guidelines in "Testing Philosophy and Guidelines" section

**Reference examples:**
- Tier 1: Most current tests (~50-100 lines)
- Tier 2: `test/test_format.cpp` (273 lines), `test/test_vector.cpp` (~390 lines)

#### 4. Update Documentation

Update **only** the "Available Modules" table in `README.md`:

- Add a new row with: header name, module import statement, status (✅ or ⚠️), and notes if applicable
- Follow the existing pattern exactly
- **Do NOT** add the module anywhere else in README.md (no exhaustive lists per the README philosophy)
- The table is around line 137-192 in the current README.md

#### 5. Test the Implementation

```bash
# Clean build
rm -rf build

# Configure with only the new module
cmake -B build -G Ninja \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DSTD_MODULE_BUILD_ALL_MODULES=OFF \
  -DSTD_MODULE_BUILD_HEADER=ON

# Build and test
cmake --build build
ctest --test-dir build --output-on-failure
```

#### 6. Handle C++20 Module ADL Limitations - **SOLUTION FOUND** ✅

**BREAKTHROUGH DISCOVERY (2025-11-14):** The C++20 module ADL limitation CAN be fixed by
explicitly exporting operators in the module!

**The Problem:**
- Argument-Dependent Lookup (ADL) doesn't work across module boundaries
- Operators like `operator<<` aren't found even when types are exported
- Without includes, string output shows memory addresses instead of text
- Affects: stream operators, manipulators, arithmetic operators, etc.

**THE SOLUTION:**
Export operators explicitly in your module:

```cpp
export module std_module.header;

export namespace std {
    // Export types/objects
    using std::cout;
    using std::vector;

    // CRITICAL: Export operators to fix ADL
    using std::operator<<;
    using std::operator>>;
    using std::operator+;
    using std::operator-;
    // ... export all relevant operators

    // Export manipulators if needed
    using std::endl;
    using std::flush;
}
```

**Verification:**
After exporting operators, test with **NO `#include` directives** - only `import`:

```cpp
import std_module.header;
// NO #include directives!

int main() {
    std::cout << "Hello World" << std::endl;  // Works!
}
```

**DO NOT (anymore):**
- ❌ Add `#include <header>` to the test file to work around the issue
- ❌ Accept broken operators as "unfixable" - they ARE fixable
- ❌ Mix `import` + `#include` of the same header (causes different issues)

**REQUIRED ACTIONS (UPDATED 2025-11-14):**

1. **Export operators in the module (.cppm file):**
   ```cpp
   export namespace std {
       // Export the types/objects
       using std::cout;

       // CRITICAL: Export ALL relevant operators
       using std::operator<<;
       using std::operator>>;
       using std::operator+;
       using std::operator-;
       // ... etc

       // Export manipulators if needed
       using std::endl;
       using std::flush;
   }
   ```

2. **Test with NO `#include` directives:**
   - Remove ALL `#include` statements from test file
   - Use ONLY `import std_module.header;`
   - This validates true module isolation
   - See `test/test_iostream.cpp` for reference

3. **If tests fail, add more operator exports:**
   - Identify which operators are missing from error messages
   - Add them to the module's export list
   - Rebuild and retest

4. **Verify on multiple compilers/stdlibs:**
   ```bash
   # Test with Clang + libstdc++
   cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++
   cmake --build build && ctest --test-dir build

   # Test with Clang + libc++
   cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++ \
     -DCMAKE_CXX_FLAGS="-stdlib=libc++" \
     -DCMAKE_EXE_LINKER_FLAGS="-stdlib=libc++ -lc++abi"
   cmake --build build && ctest --test-dir build

   # Test with GCC
   cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=g++-14
   cmake --build build && ctest --test-dir build
   ```

5. **Update the "Affected Modules List":**
   - Add to **✅ FIXED** section when operators work
   - Document which operators were exported
   - Note any remaining limitations

**Example - iostream module (FIXED):**
- **Solution:** Export `operator<<`, `operator>>`, `endl`, `flush`
- **Impact:** 100% functionality restored
- **Status:** Works on all tested compilers (Clang+libstdc++, Clang+libc++, GCC+libstdc++)
- **Reference:** `src/iostream.cppm`, `test/test_iostream.cpp`

### Testing Philosophy and Guidelines

**Test infrastructure location:** `test/`

#### Core Testing Principle

**What we test:** Module integration and C++20 module compatibility, NOT standard library correctness.

- ✅ Verify `import std_module.vector;` works
- ✅ Verify symbols are accessible through the module
- ✅ Verify ADL-dependent operators work (or document as broken)
- ❌ Don't test `std::vector::push_back()` correctness (trust the stdlib)
- ❌ Don't exhaustively test edge cases (stdlib's job)

#### The Three Testing Tiers

| Tier | Target Lines | When to Use | What to Test |
|------|--------------|-------------|--------------|
| **Tier 1: Minimal** | 50-100 | Simple utilities, rarely-used modules | Basic usage, iterators, ADL operators (status only) |
| **Tier 2: Comprehensive** | 200-400 | Core containers, I/O, algorithms | All operations categories, extensive ADL testing |
| **Tier 3: Integration** | Future | Cross-module dependencies | Wait until dependent modules available |

**Tier 1 Template (Minimal Viable Test):**

```cpp
import std_module.header;
#include <iostream>  // Allowed - testing infrastructure
#include <cassert>   // Allowed - assert is a macro

void test_basic_usage() {
    std::cout << "Testing basic usage...\n";
    std::container<int> c{1, 2, 3};
    assert(c.size() == 3);
    std::cout << "  ✓ Construction and size\n";
}

void test_iterators() {
    std::cout << "\nTesting iterators...\n";
    std::container<int> c{1, 2, 3};
    int sum = 0;
    for (int val : c) sum += val;
    assert(sum == 6);
    std::cout << "  ✓ Range-based for\n";
}

void test_adl_operators() {
    std::cout << "\nTesting ADL operators...\n";
    try {
        bool eq = (a == b);
        std::cout << "  ✓ operator== works\n";
    } catch (...) {
        std::cout << "  ⚠ operator== unavailable (ADL limitation)\n";
    }
}

int main() {
    std::cout << "=== Testing std_module.header ===\n\n";
    test_basic_usage();
    test_iterators();
    test_adl_operators();
    std::cout << "\n=== All tests passed! ===\n";
    return 0;
}
```

**Tier 2:** Add tests for all operation categories (construction, modifiers, accessors, capacity, algorithms). See `test/test_format.cpp` or `test/test_vector.cpp` for examples.

**Tier 3:** Comment out tests requiring other modules:
```cpp
// FUTURE: Requires import std_module.string
// void test_with_strings() { ... }
```

#### Dependency Policy

**Allowed `#include` directives:**

1. ✅ `<cassert>` - Required (assert is a macro)
2. ✅ `<iostream>` - Temporary (until `std_module.iostream` is universal)
3. ✅ `<sstream>` - Temporary (for string stream testing)
4. ❌ Everything else - Wait for modules or document as FUTURE

**CRITICAL:** Never `#include <header>` when testing `import std_module.header;`

```cpp
// ✅ CORRECT - Pure module testing
import std_module.vector;
#include <iostream>  // OK - testing infrastructure
#include <cassert>   // OK - testing infrastructure

// ❌ WRONG - Defeats purpose of module testing
import std_module.vector;
#include <vector>  // NO! This smuggles in the header
```

#### ADL Operator Testing (Required for All Tests)

All tests must attempt ADL-dependent operators and report status without failing:

```cpp
void test_operators() {
    std::cout << "\nTesting operators...\n";

    try {
        bool result = (a == b);  // Try the operator
        assert(result);
        std::cout << "  ✓ operator== works\n";
    } catch (...) {
        std::cout << "  ⚠ operator== unavailable (ADL limitation)\n";
        // Don't fail the test - this is a known C++20 module issue
    }
}
```

**Important:** Never fail tests due to ADL limitations. Document them and continue. See the "C++20 Module ADL Limitation" section below for the solution (export operators explicitly).

#### When to Use Each Tier

| Module Type | Tier | Rationale |
|-------------|------|-----------|
| Core containers (`vector`, `map`, `string`) | 2 | Heavily used, worth comprehensive testing |
| Simple utilities (`limits`, `bit`, `compare`) | 1 | Small API, minimal test sufficient |
| I/O modules (`iostream`, `fstream`) | 2 | ADL-heavy, need extensive operator testing |
| Algorithms (`algorithm`, `ranges`) | 2 | Many functions, broad coverage needed |
| Type traits (`type_traits`, `concepts`) | 1 | Compile-time only, simple verification |
| Concurrency (`thread`, `mutex`, `future`) | 2 | Complex interactions, thorough testing |
| Rarely used (`codecvt`, `locale`) | 1 | Low priority, minimal test acceptable |

#### Test Quality Checklist

Before committing, verify:

- [ ] Uses `import std_module.<module>;` not `#include <header>`
- [ ] Only includes allowed headers (`<cassert>`, `<iostream>`, `<sstream>`)
- [ ] Tests basic usage (construction, size, access)
- [ ] Tests iterators (if applicable) including range-based for
- [ ] Tests ADL operators with try/catch pattern (reports status, doesn't fail)
- [ ] Documents FUTURE tests requiring unavailable modules
- [ ] Returns 0 on success, 1 on failure
- [ ] Has meaningful output with ✓ checkmarks
- [ ] Appropriate line count: 50-100 (Tier 1) or 200-400 (Tier 2)
- [ ] Passes: `ctest --test-dir build -R test_<module>`
- [ ] Passes: `python3 scripts/symbol_coverage.py <module>`

#### Automated Testing Commands

```bash
# Run all tests
ctest --test-dir build --output-on-failure

# Run specific test
ctest --test-dir build -R test_format --output-on-failure

# Verbose output
ctest --test-dir build --verbose

# Check symbol coverage for a module
python3 scripts/symbol_coverage.py format

# Check coverage for all modules
python3 scripts/symbol_coverage.py --all
```

#### Symbol Coverage vs Functional Testing

| Tool | Purpose | What it Catches | What it Misses |
|------|---------|-----------------|----------------|
| `symbol_coverage.py` | Smoke test | Untested exports | Whether symbols work |
| Functional tests | Integration test | Module compilation, ADL issues, runtime errors | Nothing (comprehensive) |
| `validate_exports.py` | Meta-validation | Regex extraction errors | Export usability |

**Workflow:**
1. Write the module (`src/header.cppm`)
2. Write Tier 1 test (`test/test_header.cpp`)
3. Run functional test: `ctest --test-dir build -R test_header`
4. Check coverage: `python3 scripts/symbol_coverage.py header`
5. Add tests for uncovered symbols or document why not tested
6. Verify extraction: `python3 scripts/validate_exports.py src/header.cppm`

#### Testing Conventions

**Standard patterns from `test/test_format.cpp:1-273`:**

1. **One test function per feature area**
   ```cpp
   void test_basic_format() { /* ... */ }
   void test_format_to() { /* ... */ }
   void test_custom_formatter() { /* ... */ }
   ```

2. **Assertion-based validation**
   ```cpp
   auto result = std::format("Value: {}", 42);
   assert(result == "Value: 42");
   ```

3. **Visual feedback with checkmarks**
   ```cpp
   std::cout << "  ✓ Test description: " << result << "\n";
   ```

4. **Exception handling (never fail on expected errors)**
   ```cpp
   try {
       std::format("{:invalid}", 42);
       assert(false && "Should have thrown");
   } catch (const std::format_error& e) {
       std::cout << "  ✓ Expected error: " << e.what() << "\n";
   }
   ```

5. **Test variety, not exhaustiveness**
   - Test ONE example per operation category
   - Don't test stdlib edge cases (empty, max size, etc.)
   - Focus on module integration, not stdlib correctness

### Git Workflow and Commit Conventions

#### Commit Message Style

Based on git history:

```
48014f4 - Merge pull request #2 from ...
d4d4e27 - Complete C++20 format module implementation with comprehensive tests
0552509 - Merge pull request #1 from ...
ca8696e - Add flexible CMake build system for C++20 modules
8d9132d - Add minimal test infrastructure for manual module compilation
```

**Pattern:** Imperative mood, descriptive, mentions key features

**Examples:**
- ✅ "Add vector module with iterator support"
- ✅ "Complete C++20 string module implementation with comprehensive tests"
- ✅ "Fix alignment handling in custom formatter"
- ❌ "Added some stuff"
- ❌ "WIP"

#### Branch Strategy

- Feature branches merged via pull requests
- Branch naming: `claude/{description}-{session-id}`
- Clean, linear history preferred

#### Push Requirements

**CRITICAL:** When pushing, always use the correct branch format:

```bash
git push -u origin claude/{description}-{session-id}
```

**Requirements:**
- Branch MUST start with `claude/`
- Branch MUST end with the matching session ID provided by the system
- Use `-u` flag for upstream tracking
- Retry on network errors (up to 4 times with exponential backoff: 2s, 4s, 8s, 16s)

**Example:**
```bash
git push -u origin claude/add-chrono-module-01H9G7pceQwCzk5zrUd8c8M1
```

## Common Pitfalls and Important Notes

### Build System

1. **Never use Make generator** - C++20 modules require Ninja
   ```bash
   # ❌ WRONG
   cmake -B build

   # ✅ CORRECT
   cmake -B build -G Ninja
   ```

2. **CMake version matters** - 3.28+ required for `FILE_SET CXX_MODULES`
   ```cmake
   # Check: CMakeLists.txt:1
   cmake_minimum_required(VERSION 3.28)
   ```

3. **Module files need special handling**
   ```cmake
   # ❌ WRONG - treats as regular source
   target_sources(target PRIVATE module.cppm)

   # ✅ CORRECT - handles as module
   target_sources(target PUBLIC FILE_SET CXX_MODULES FILES module.cppm)
   ```

### Module Implementation

1. **Global module fragment required for headers**
   ```cpp
   // ✅ CORRECT
   module;
   #include <format>
   export module std_module.format;

   // ❌ WRONG - include after module declaration
   export module std_module.format;
   #include <format>
   ```

2. **Export the namespace, not individual symbols**
   ```cpp
   // ✅ CORRECT - exports under std namespace
   export namespace std {
       using std::format;
   }

   // ❌ WRONG - pollutes global namespace
   export using std::format;
   ```

3. **Module names use dots, not colons**
   ```cpp
   // ✅ CORRECT
   export module std_module.format;

   // ❌ WRONG
   export module std_module::format;
   ```

### Testing

1. **Module import must come first**
   ```cpp
   // ✅ CORRECT
   import std_module.format;
   #include <iostream>

   // ❌ WRONG - imports after includes
   #include <iostream>
   import std_module.format;
   ```

2. **Tests must use ONLY `import` - NO `#include` of the wrapped header**
   - **Critical Rule:** Tests should rely exclusively on `import std_module.header;`
   - Do NOT add `#include <header>` as a workaround for missing functionality
   - If functionality doesn't work with just `import`, mark it as FIXME
   - This policy ensures we accurately test what the module exports

   ```cpp
   // ✅ CORRECT - Tests only module exports
   import std_module.complex;
   #include <iostream>  // OK - for testing infrastructure
   #include <cassert>   // OK - for testing infrastructure

   // ❌ WRONG - Defeats the purpose of testing the module
   import std_module.complex;
   #include <complex>  // NO! Don't work around module limitations
   ```

3. **C++20 Module ADL Limitation with Non-Member Operators - CRITICAL PROJECT-WIDE ISSUE**

   **This is an endemic problem affecting many standard library modules.**

   - **Core Issue:** Non-member operators and hidden ADL-dependent functions are not properly
     exported/found when using C++20 modules, even when using declarations export them
   - **Technical Cause:** Argument-Dependent Lookup (ADL) doesn't work correctly with modules
   - **Reference:** https://github.com/cplusplus/papers/issues/1005

   **Affected Module Categories:**

   a) **Arithmetic/Comparison Operators** (e.g., `<complex>`, `<valarray>`)
      - **Problem:** `operator+`, `operator-`, `operator*`, etc. not found
      - **Example:** `std::complex<double> c1(1,2), c2(3,4); auto sum = c1 + c2;` fails
      - **Solution:** Comment out affected tests with `// FIXME: C++20 module ADL limitation`
      - **File:** `test/test_complex.cpp` has operator tests commented out

      ```cpp
      // FIXME: C++20 module ADL limitation - operators not found without #include
      // void test_arithmetic_operations() {
      //     std::complex<double> c1(3.0, 4.0);
      //     auto sum = c1 + c2;  // Error: operator+ not found via ADL
      // }
      ```

   b) **I/O Manipulator Operators** (e.g., `<iomanip>`)
      - **Problem:** `operator<<` overloads for manipulator return types not found
      - **Details:** Manipulators like `std::setw(10)` return hidden types (e.g., `std::_Setw`),
        and the corresponding `operator<<(ostream&, _Setw)` overloads are not exported/found
      - **Impact:** Cannot use manipulators with streams when only importing the module
      - **Example:** `std::cout << std::setw(10) << 42;` fails to compile
      - **Root Cause:** The return types are implementation details, and their operators aren't
        found via ADL through module boundaries
      - **Status:** Currently blocks `<iomanip>` module implementation
      - **Workaround:** Tests must `#include <iomanip>` in addition to `import std_module.iomanip;`
        which defeats the purpose of testing the module in isolation

      ```cpp
      // Example failure:
      import std_module.iomanip;
      #include <iostream>
      #include <sstream>

      std::ostringstream oss;
      oss << std::setw(10) << 42;  // Error: no operator<<(ostream&, _Setw)
      ```

   c) **Stream Insertion/Extraction Operators** (potentially affects many types)
      - **Problem:** Similar issues may affect custom `operator<<` and `operator>>` overloads
      - **Impact:** May affect modules like `<chrono>`, `<filesystem>`, `<thread>`, etc.
      - **Status:** Needs investigation as more modules are implemented

   **General Guidelines:**

   1. **When implementing a new module:** Test ALL operators thoroughly
   2. **When operators fail:** Document with `// FIXME: C++20 module ADL limitation`
   3. **Don't work around in tests:** Do NOT add `#include <header>` to fix operator issues
      - Adding the include defeats the purpose of testing module isolation
      - It hides the limitation, making users think the module is complete
   4. **Mark as known limitation:** Update module documentation to note which features don't work
   5. **Track affected modules:** Maintain a list of partially functional modules

   **Affected Modules List:**

   **✅ FIXED (operator exports work):**
   - ✅ `<iostream>` - **FIXED** by exporting `operator<<`, `operator>>`, `endl`, `flush`
     - Reference: `src/iostream.cppm`, `test/test_iostream.cpp`
     - Works on Clang + libstdc++, Clang + libc++, GCC + libstdc++

   - ✅ `<chrono>` - **FIXED** by exporting arithmetic, comparison, and literal operators
     - Exported operators: `+`, `-`, `*`, `/`, `%`, `==`, `<`, `<=`, `>`, `>=`, `<=>`
     - Exported literals: `operator""h`, `operator""min`, `operator""s`, etc. in inline namespace
     - Calendar operators (operator/) work via re-export in std namespace
     - Stream operators (`operator<<`, `operator>>`) exported
     - **Outstanding issue:** `std::exception` not available from chrono module alone
       - Workaround: `#include <exception>` in test file
       - This is expected - exception types from `<chrono>` should import from std_module.exception
     - Reference: `src/chrono.cppm`, `test/test_chrono.cpp`
     - Works on Clang + libstdc++, Clang + libc++, GCC + libstdc++

   **⚠️ TODO (not yet tested with operator exports):**
   - ⚠️ `<iomanip>` - May be fixable by exporting manipulator operators
   - ⚠️ `<complex>` - Should be fixable by exporting arithmetic operators
   - ⚠️ Potentially fixable: `<valarray>`, `<filesystem>`, `<thread>`, etc.

   **SOLUTION PATTERN:**
   - ✅ Export operators explicitly: `using std::operator<<;`
   - ✅ Export manipulators/literals explicitly: `using std::endl;`
   - ✅ Export literals in inline namespace for proper accessibility
   - ✅ Test with NO `#include` directives (except for required workarounds)
   - ✅ Works across all practical compiler/stdlib combinations

4. **Test all exported symbols** - Easy to miss symbols in export list
   - Check the module file for complete export list
   - Example: `src/format.cppm:13-46` exports 20+ symbols
   - Mark untestable functions with FIXME if they require broken operators

5. **Custom formatters need template specialization**
   ```cpp
   // From test/test_format.cpp:20-29
   template<>
   struct std::formatter<Point> {
       // ...
   };
   ```

### File Organization

1. **Module files use .cppm extension** - Not .cpp or .cxx
2. **One module per file** - Don't combine multiple modules
3. **Test files match module names** - `test_{module}.cpp` pattern

## Quick Reference

### File Path Reference

| Component | Path |
|-----------|------|
| Root build config | `/home/user/std_module/CMakeLists.txt` |
| Module implementations | `/home/user/std_module/src/*.cppm` |
| Module build config | `/home/user/std_module/src/CMakeLists.txt` |
| Test files | `/home/user/std_module/test/test_*.cpp` |
| Test build config | `/home/user/std_module/test/CMakeLists.txt` |
| Package config template | `/home/user/std_module/cmake/std_module-config.cmake.in` |

### Command Quick Reference

```bash
# Clean build from scratch
rm -rf build && cmake -B build -G Ninja -DCMAKE_CXX_COMPILER=clang++

# Build specific module only
cmake -B build -G Ninja -DSTD_MODULE_BUILD_ALL_MODULES=OFF -DSTD_MODULE_BUILD_FORMAT=ON

# Run tests with verbose output
ctest --test-dir build --output-on-failure --verbose

# Check what targets are available
cmake --build build --target help

# Install to custom location
cmake --install build --prefix ~/.local
```

### Integration Examples

**Method 1: Subdirectory**
```cmake
add_subdirectory(external/std_module)
target_link_libraries(myapp PRIVATE std_module::format)
```

**Method 2: Installed Package**
```cmake
find_package(std_module REQUIRED COMPONENTS format)
target_link_libraries(myapp PRIVATE std_module::format)
```

**Method 3: Manual (see test/README.md)**

## Code Quality Standards

### When Writing Module Wrappers

1. **Export completeness** - Export ALL public APIs from the header
   - Check cppreference.com for complete API list
   - Include functions, classes, type aliases, constants

2. **Documentation** - Add brief file header
   ```cpp
   /**
    * @file format.cppm
    * @brief C++20 format module wrapper
    */
   ```

3. **Consistency** - Follow format.cppm pattern exactly
   - Same structure, same ordering
   - Global module fragment → export module → export namespace

### When Writing Tests

Follow the **Testing Philosophy and Guidelines** section for comprehensive instructions.

**Key principles:**
1. **Test module integration, not stdlib correctness** - Trust the standard library works
2. **Choose appropriate tier** - Tier 1 (50-100 lines) for simple modules, Tier 2 (200-400 lines) for important ones
3. **Pure module testing** - Only `import std_module.header;`, never `#include <header>`
4. **Allowed includes** - Only `<cassert>`, `<iostream>`, `<sstream>` for testing infrastructure
5. **ADL operator testing** - Always test operators with try/catch, document status, never fail
6. **Self-contained functions** - Each test function is independent
7. **Readable output** - Use checkmarks (✓) and descriptions
8. **Document dependencies** - Comment out FUTURE tests requiring unavailable modules

**Reference implementations:**
- Tier 1: Most current tests (~50-100 lines, minimal viable)
- Tier 2: `test/test_format.cpp` (273 lines), `test/test_vector.cpp` (390 lines, comprehensive)

## Security Considerations

This is a wrapper library that re-exports standard library symbols:

- **No network operations** - Pure local compilation
- **No file I/O** - Except during build/install
- **No external dependencies** - Only standard library
- **No unsafe operations** - Just wrapping existing stdlib APIs

## Performance Notes

1. **Module compilation is slower initially** - But faster for consumers
2. **Precompiled modules (.pcm) enable faster builds** - Cached by CMake
3. **No runtime overhead** - Same performance as `#include`

## Future Expansion Roadmap

**Status:** 72 of 90+ standard library headers wrapped

**Remaining High-Priority Candidates:**

Most commonly-used C++ standard library headers have been implemented. The remaining ~18-20 headers fall into these categories:

1. **C++23 New Features:**
   - `<print>` - Modern printing facilities
   - `<expected>` - Error handling type
   - `<mdspan>` - Multidimensional span
   - `<generator>` - Coroutine generators
   - `<flat_map>`, `<flat_set>` - Flat associative containers
   - `<stacktrace>` - Stack trace library
   - `<stdfloat>` - Fixed-width floating-point types

2. **Utility Headers:**
   - `<utility>` - General utilities (pair, move, forward, etc.)
   - `<functional>` (already implemented, verify completeness)

3. **C Library Wrappers:**
   - `<cmath>`, `<cstdio>`, `<cstring>`, `<cstdlib>`, `<cctype>`
   - `<cwchar>`, `<cwctype>`, `<cuchar>` - Wide character support
   - `<ctime>` - C time library
   - `<csignal>`, `<csetjmp>` - Signal handling and non-local jumps
   - `<cfenv>`, `<clocale>` - Floating-point environment and locale

4. **Specialized/Rarely-Used:**
   - Various deprecated or platform-specific headers

**Note:** The C library wrapper headers (`<c*>`) may require special handling since they export C symbols, not C++ symbols.

**Aggregate Target:**

The `std_module::all` CMake target already provides a way to link all modules at once:
```cmake
target_link_libraries(myapp PRIVATE std_module::all)
```

**ADL Limitation Tracking:**

Continue to document modules affected by C++20 ADL issues. Status of known cases:

**✅ RESOLVED:**
- ✅ `<iostream>` - Fixed by exporting stream operators
- ✅ `<chrono>` - Fixed by exporting arithmetic, comparison, and literal operators

**⚠️ PARTIALLY FUNCTIONAL:**
- ⚠️ `<iomanip>` - Manipulator operators still problematic (implementation-specific types)
- ⚠️ `<complex>` - Arithmetic operators need export fix (TODO: apply operator export solution)

**🔄 TODO (apply operator export solution):**
- `<valarray>` - Should be fixable by exporting arithmetic operators
- `<filesystem>` - Should be fixable by exporting path operators

**Solution:** Export all relevant operators explicitly in the module. See `src/iostream.cppm` and `src/chrono.cppm` for reference patterns.

## Troubleshooting Guide

### Build Issues

**Problem:** "CMake 3.28 or higher is required"
- **Solution:** Update CMake or use newer compiler's bundled CMake

**Problem:** "Ninja not found"
- **Solution:** Install ninja-build package: `apt-get install ninja-build`

**Problem:** Module compilation fails with cryptic errors
- **Solution:** Check compiler version (Clang 16+), verify Ninja generator is used

### Test Failures

**Problem:** Test compiles but crashes
- **Solution:** Check that ALL used symbols are exported in the module

**Problem:** Linker errors about undefined symbols
- **Solution:** Verify target links against module target (`std_module::format`)

### Git Issues

**Problem:** Push fails with 403
- **Solution:** Verify branch name starts with `claude/` and ends with session ID

## Version History

- **0.1.0** (Current) - Production-ready release with 72 modules
  - **Module System:**
    - 72 standard library modules with comprehensive tests
    - Macro-based CMake infrastructure for easy module addition
    - Symbol coverage analysis tool
  - **Build System:**
    - Flexible CMake 3.28+ build system with Ninja generator
    - Per-module build options (`STD_MODULE_BUILD_<NAME>`)
    - Aggregate `std_module::all` target
    - Full installation support
  - **Integration:**
    - Multiple integration methods (subdirectory, installed, manual)
    - Package config for find_package() support
  - **Testing:**
    - Comprehensive test suite for all modules
    - Automated symbol coverage analysis
    - CTest integration
  - **Documentation:**
    - Comprehensive CLAUDE.md for AI assistants (detailed, pattern-focused)
    - Lean, user-facing README with pattern-focused documentation
    - Single source of truth: "Available Modules" table lists all 72 modules
    - C++20 module ADL limitation documentation and solutions
    - Testing philosophy and guidelines (three-tier system)

**Implemented Modules (72):**
algorithm, any, array, atomic, barrier, bit, bitset, charconv, chrono, codecvt, compare, complex, concepts, condition_variable, coroutine, deque, exception, execution, filesystem, format, forward_list, fstream, functional, future, initializer_list, iomanip, ios, iosfwd, iostream, istream, iterator, latch, limits, list, locale, map, memory, memory_resource, mutex, new, numbers, numeric, optional, ostream, queue, random, ranges, ratio, regex, scoped_allocator, semaphore, set, source_location, span, stack, stdexcept, stop_token, streambuf, string, string_view, syncstream, system_error, thread, tuple, type_traits, typeindex, typeinfo, unordered_map, unordered_set, valarray, variant, vector

---

**Last Updated:** 2025-11-15
**Repository:** /home/user/std_module
**Primary Maintainer:** AI Assistant
**Documentation Version:** 3.2 (Updated module count and roadmap)

---
> Source: [whatDiracIsCooking/std_module](https://github.com/whatDiracIsCooking/std_module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
