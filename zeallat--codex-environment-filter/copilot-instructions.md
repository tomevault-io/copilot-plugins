## codex-environment-filter

> - Use `npm run test` to execute all tests defined across all packages.

# Contributor Guide

## Testing Instructions

- Use `npm run test` to execute all tests defined across all packages.
- Use `npm run test -- --filter <project_name>` to execute all tests defined in the specified package.
- Use `npm run typecheck` to perform type checking across all packages.
- Use `npm run typecheck -- --filter <project_name>` to perform type checking in the specified package.
- Use `npm run lint` to execute ESLint checks across all packages.
- Use `npm run lint -- --filter <project_name>` to execute ESLint checks in the specified package.
- Always run all tests and type checks from the repository root using npm scripts. This is necessary because dependency builds might rely on Turborepo's dependency graph.
- All tests must pass before merging.
- All type errors must be resolved before merging.
- Fix all tests and type errors until the entire test suite turns green.
- After moving files or changing imports, run `npm run lint` or `npm run lint -- --filter <project_name>` to ensure compliance with ESLint rules.
- Always add or update tests for any modified code, even if not explicitly requested.

## Code Guidelines

1. Choose clear, meaningful names.
2. Prefer explicit type annotations and generics for clarity and type safety.
3. Limit functions to one clear responsibility and under 80 lines.
4. Decompose complex logic into small, composable functions.
5. Embrace immutability by returning new data instances.
6. Clearly separate pure logic from side-effect operations.
7. Keep data structures distinct from manipulation logic.
8. Use reusable patterns and higher-order functions to minimize repetition.
9. Improve testability through purity and dependency injection.
10. Prefer built-in or established third-party libraries (e.g., Lodash) over custom implementations.
11. Avoid advanced functional concepts (monads, specialized libraries); prioritize maintainability.
12. Export only one item per file (function, class, type, or interface), except in barrel (`index`) files.
13. Directory names use kebab-case; filenames exactly match their exported item.
14. Library packages automatically generate barrel files during the build process; do not edit index files manually. Refer to section 'Barrel File Generation Rules' below for barrel file generation rules.

## Barrel File Generation Rules (that barrel file generator cli using. Do not generate or modify barrel file manually.)

1. Default directory: `./src`.
2. Generates or overwrites `index.ts`, exporting `.ts`, `.tsx`, and subdirectory barrels.
3. Export groups:
    - Type-only: `export { type MyType }`
    - Default types: `export { type default as SomeType }`
    - Values: exported via namespace objects named after directories.
4. Optional `--export-all`: Directly re-exports all items.
5. Namespace naming:
    - Root (`src`): Uses `--namespace` or parent directory name.
    - Subdirectories: Use their own directory names.
6. Overwrite mode: Deletes and recreates existing `index.ts` files.

## Guidelines for Writing Test Code

### Do:

- Test behavior, focusing on input/output and real-world scenarios.
- Use clear, descriptive test names.
- Ensure tests are isolated without side effects.
- Follow the Arrange-Act-Assert (AAA) structure.
- Limit to one logical assertion per test.
- Use setup/teardown carefully to avoid hidden dependencies.
- Handle asynchronous tests correctly.
- Keep tests fast and deterministic by mocking external dependencies.
- Explicitly cover edge cases and error conditions.
- Provide clear and informative assertion messages.
- Maintain one test case per file, naming the file after the test.
- When creating test files, separate each individual test case (e.g., an it block) into its own file.
- A single test code file can contain only one test (it) block. Multiple it blocks within one file are not allowed.
- Use this naming convention for test code files: `[target].[action].[expectedResult].test.ts`
- Mock only _external dependencies at the module boundary_—never internal helpers.
- Avoid spying on or asserting against private implementation details.
- Trigger error branches by controlling the environment (e.g., mock a failing API), not by rewriting internals.
- Use fake timers to simulate time-based behavior without inspecting hidden state.
- Keep unit black-box tests (fast, with boundary mocks) separate from integration tests (real dependencies).
- **Rule of thumb:** a test that’s clear to someone who knows only the public API—nothing about the code inside—is black-box.

### Don't:

- Test internal implementation details.
- Write interdependent tests.
- Include complex conditional logic.
- Overuse mocking.
- Write tests with insufficient assertions.

## PR instructions

Title format: [<project_name>] <Title>

---
> Source: [zeallat/codex-environment-filter](https://github.com/zeallat/codex-environment-filter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
