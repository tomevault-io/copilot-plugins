## windsurf

> **Preamble:** These rules define the universal coding standards for the project. Language-specific conventions can be added as sub-sections.

# 20: Coding Standards

**Preamble:** These rules define the universal coding standards for the project. Language-specific conventions can be added as sub-sections.

## General Rules
20.1. **Readability:** Code must be written to be easily understood by other human developers. Prioritize clarity over cleverness.
20.2. **DRY (Don't Repeat Yourself):** Avoid duplicating code. Use functions, classes, and modules to promote reusability.
20.3. **Comments:** Add comments to explain *why* something is done, not *what* is being done. The code itself should explain the "what".
20.4. **Error Handling:** All functions that can fail (e.g., I/O, network requests) must have robust error handling.

## Language: TypeScript/JavaScript
20.5. **Formatter:** All `.ts` and `.js` files must be formatted with Prettier using the project's `.prettierrc` configuration.
20.6. **Linter:** All code must pass ESLint checks without any errors.
20.7. **Naming:**
    - `camelCase` for variables and functions.
    - `PascalCase` for classes and types.
    - `UPPER_SNAKE_CASE` for constants.
20.8. **Imports:** Use ES6 `import/export` syntax. Imports should be sorted alphabetically.

## Language: Python
20.9. **Formatter:** All `.py` files must be formatted with Black.
20.10. **Linter:** All code must pass `ruff` checks.
20.11. **Naming:**
    - `snake_case` for variables and functions.
    - `PascalCase` for classes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rcesaret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
