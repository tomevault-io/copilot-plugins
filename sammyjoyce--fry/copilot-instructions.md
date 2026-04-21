## fry

> These rules define the development principles and practices for the fry project.


# fry

These rules define the development principles and practices for the fry project.

## Core Development Guidelines

<development_guidelines>

- Before making changes, always search the codebase - don't assume functionality isn't implemented
- After implementing functionality or resolving problems, run the tests for that unit of code
- If tests unrelated to your work fail, it's your job to resolve these as part of the change
- Keep AGENTS.md up to date with information on how to build the compiler and optimization learnings
- DO NOT IMPLEMENT PLACEHOLDER OR SIMPLE IMPLEMENTATIONS - FULL IMPLEMENTATIONS ONLY
- When you discover a bug, resolve it using subagents even if it is unrelated to the current piece of work
- Zero technical debt policy - do it right the first time
- Zero dependencies policy - no external dependencies beyond build tools
- Single sources of truth - no migrations/adapters
- Learning documentation - update AGENTS.md with build/test learnings
- No status reports - DO NOT PLACE STATUS REPORT UPDATES INTO AGENTS.md
</development_guidelines>

## Safety First Principles

<safety_principles>

- No recursion - use iterative approaches with explicit bounds
- Limit everything - all loops must have fixed upper bounds
- Assert extensively - average 2+ assertions per function minimum, use static_assert
- Pair assertions - assert properties in at least two code paths
- Assert positive AND negative space - check what you expect AND what you don't
- Static memory allocation - allocate all memory at startup, no dynamic allocation after init
- Fail fast - detect violations immediately rather than later
- Handle all errors - every error must be explicitly handled
</safety_principles>

## Code Organization Standards

<code_organization>

- 70 lines max per function - hard limit, no exceptions
- Smallest possible scope - minimize variables in scope
- Control flow clarity - keep all switch/if in parent functions, pure logic in helpers
- Explicit over implicit - pass all options explicitly, don't rely on defaults
- Always say why - comments explain rationale, not just what
- Perfect names matter - take time to find the right nouns and verbs
- No abbreviations - use full descriptive names (except loop counters)
- Units last - use naming like `latency_ms_max` not `max_latency_ms`
- Same-length related names - use `source`/`target` not `src`/`dest`
- Big-endian naming - most significant word first
- Order matters - fields → Types → Methods in structs
</code_organization>

## Development Practices

<development_practices>

- Zero technical debt policy - do it right the first time
- Zero dependencies - no external dependencies beyond build tools
- Compound assertions - split `assert(a && b)` into `assert(a); assert(b)`
- In-place construction - initialize large structs in-place with out pointers
- Calculate near use - don't introduce variables before needed
</development_practices>

## Performance Optimization

<performance>
- Design-phase optimization - best performance wins come from architecture
- Back-of-envelope sketches - always estimate resource usage (network/disk/memory/CPU)
- Batch operations - amortize costs across multiple operations
- Hot loop extraction - extract performance-critical loops into standalone functions
- Use arena allocation for performance-critical paths
</performance>

## Naming & Style

<naming_style>

- Perfect names matter - take time to find the right nouns and verbs
- No abbreviations - use full descriptive names (except loop counters)
- Units last - use naming like `latency_ms_max` not `max_latency_ms`
- Same-length related names - use `source`/`target` not `src`/`dest`
- Big-endian naming - most significant word first
- Order matters - fields → Types → Methods in structs
</naming_style>

## C Language Standards

<c_standards>

- C23 with `-D_GNU_SOURCE` for POSIX extensions
- Google style via .clang-format (2-space indent, 80 col limit)
- Headers: Use `#pragma once`, group includes: system → project → local
- Error Handling: Return `app_error` enum values, never raw integers
- Memory: Use `app_malloc/app_free` wrappers from `utils/memory.h`
- Logging: Use `LOG_*` macros from `utils/logging.h` (DEBUG/INFO/WARN/ERROR)
- Naming: snake_case for functions/variables, UPPER_CASE for macros/enums
- Types: Suffix custom types with `_t` (e.g., `app_config_t`)
- Functions: Prefix with module name (e.g., `app_config_create`)
- Attributes: Use `APP_NODISCARD` for functions that return errors
- TUI: Conditionally compiled with `-DENABLE_TUI=1` flag
</c_standards>

## Build & Test Commands

<build_test_commands>

- Build: `zig build` (debug) or `zig build -Doptimize=ReleaseSafe` (release)
- Run tests: `zig build test` or `just test`
- Run single test: Tests are in `test/main.zig` - modify and run `zig build test`
- Format check: `zig build fmt-check` or `clang-format --dry-run src/**/*.{c,h}`
- Lint: Use pre-commit hooks: `pre-commit run --all-files`
- Sign binary (macOS): `zig build sign` for ad-hoc signing or `zig build -Dsign-identity="Developer ID" sign-cert` for certificate signing
  - Required on macOS 15+ for keychain access, otherwise falls back to in-memory storage
</build_test_commands>

## Modern C Patterns & Best Practices

<modern_c_patterns>

### Error Handling Pattern

Use the `goto cleanup` idiom for centralized resource cleanup:

```c
int process_file(const char *filename) {
    FILE *file = NULL;
    char *buffer = NULL;
    int status = -1;

    file = fopen(filename, "r");
    if (!file) goto cleanup;

    buffer = app_malloc(1024);
    if (!buffer) goto cleanup;

    // Process file...
    status = 0;

cleanup:
    if (buffer) app_free(buffer);
    if (file) fclose(file);
    return status;
}
```

### Object-Oriented Patterns

Simulate OOP using structs with function pointers:

```c
typedef struct {
    // Data members
    void *data;
    size_t size;

    // Methods
    void (*destroy)(void *self);
    int (*process)(void *self, const char *input);
} app_handler_t;
```

### Memory Management

- Arena Allocation: For performance-critical paths, use arena allocators
- Ownership: Be explicit about memory ownership in function documentation
- Validation: Always check malloc returns and validate sizes before allocation

### Concurrency (C11)

- Use `_Atomic` types for lock-free counters
- Protect shared data with `mtx_t` mutexes
- Use condition variables (`cnd_t`) for thread coordination
- Follow the pattern: lock → while(condition) wait → work → unlock

### Security Practices

- Input Validation: Sanitize all external input, especially sizes and indices
- Bounds Checking: Use sized functions (snprintf, strncat, etc.)
- Integer Overflow: Use C23's `<stdckdint.h>` for safe arithmetic when available
- Compiler Hardening: Default flags include `-fstack-protector-strong -D_FORTIFY_SOURCE=2`

### Type Safety

- Use `static_assert` for compile-time invariants
- Prefer typed enums over raw integers for state/status
- Use opaque pointers to hide implementation details
- Apply `const` correctly for immutable data

### Modern C Features (C11/C23)

- `_Generic` for type-generic macros
- `static_assert` for compile-time checks
- `alignas` for explicit alignment requirements
- Anonymous structs/unions for cleaner APIs
- `typeof` for type inference in macros (C23)

</modern_c_patterns>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sammyjoyce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
