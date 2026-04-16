## cp

> Run formatting then tests automatically - do not wait to be asked:

# CLAUDE.md - Rules for this repository

## After editing C++ files

Run formatting then tests automatically - do not wait to be asked:

```bash
{ git diff --name-only --diff-filter=ACM; git ls-files --others --exclude-standard; } | \
  grep -E '\.(cpp|hpp|h|cc|cxx)$' | \
  xargs -r clang-format-19 -i
make test
```

Use `clang-format-19` specifically - the system default (`clang-format`) is v18, which
lacks `BreakTemplateDeclarations` support.

## Naming conventions

| Kind                                      | Style        |
|-------------------------------------------|--------------|
| Class / Struct names                      | `PascalCase` |
| Function / Method names                   | `snake_case` |
| Variables / Parameters / Members / Files  | `snake_case` |

## Comment style

- Own-line comments that are full sentences must end with `.`
- Own-line comments that are labels or headings: no period
- Inline end-of-line comments: no period

## Writing style

- Use `-` (hyphen), not `—` (long dash)
- Use `->` (ASCII arrow), not `→` (Unicode arrow)

## Code review

When asked to review C++ code, focus on: correctness, undefined behavior, convention
violations, and performance pitfalls. Be direct and specific, reference line numbers
where relevant. Assume competitive programming context where some shortcuts are
acceptable.

## Git commits

- Do not add `Co-Authored-By: Claude` lines to commits.

## Rules

1. **No external dependencies** - STL only.
2. **One class/concept per header file** - keep headers focused.
3. **Complexity docs** - every public function must have a comment noting time and space
   complexity (e.g., `// O(n log n) time, O(n) space`).
4. **All headers** - must start with `#pragma once` and wrap all code in
   `namespace cp { }`.
5. **All includes** - use rooted paths: `#include "cp/..."` (never relative `../` from
   library code).
6. **Namespace** - flat `namespace cp` only; no nested namespaces. `using namespace std`
   is declared inside `namespace cp` in `common.hpp` - no `std::` prefix needed.
7. **Struct member order** - members first, then constructors, then methods.
8. **Braces** - always use `{}` for control flow bodies, even single-line.
9. **Test every module** - add a corresponding file under `tests/<category>/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nhtrnm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
