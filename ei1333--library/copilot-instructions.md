## library

> These instructions apply to Codex throughout this repository. This project is a

# Repository Instructions for Codex

## Scope

These instructions apply to Codex throughout this repository. This project is a
C++17 library for competitive programming.

Codex may add algorithms, write or update documentation, and refactor existing
implementations. Keep each task focused on one feature or one coherent change.

## Priorities

- Prioritize runtime performance and appropriate asymptotic complexity.
- Preserve readability. Do not introduce a substantially less readable
  implementation for a minor performance improvement.
- Follow the design, naming, file placement, templates, and API conventions of
  nearby code. Prefer consistency with an existing API over inventing a new
  convention.
- Keep competitive-programming use in mind: implementations should be
  self-contained, efficient, and practical to include from a solution.
- Briefly explain non-obvious optimizations, invariants, or constraints in code
  when they would otherwise be difficult to maintain.

## Implementation Rules

- Use C++17 unless the repository is deliberately migrated in a separate task.
- Inspect related headers, tests, and documentation before changing an API.
- Preserve existing APIs and behavior. Avoid breaking changes; if a requested
  task appears to require one, stop and ask the user before implementing it.
- Do not delete files unless the user explicitly requests the deletion.
- Avoid unrelated cleanup or broad refactoring.
- Format changed C++ code with `clang-format` using the predefined Google style.
  Every changed `.hpp` file must pass the same check as CI:

  ```console
  clang-format-22 -style=google --dry-run --Werror path/to/file.hpp
  ```

- New `.hpp` files must be self-contained: directly include every standard and
  project header they depend on, qualify standard library names with `std::`,
  and compile when included first in an otherwise empty translation unit. Do
  not write `using namespace std;` in a header or rely on an includer declaring
  it first. Apply the same rules when deliberately migrating an existing
  header; migrate other existing headers incrementally when the task scope
  permits. CI automatically compiles every tracked `.hpp` except
  `template/template.hpp` in isolation.
- Do not include `template/template.hpp` from new `test/verify/*.test.cpp`
  files. Include the required standard library headers and declare any aliases
  or helper functions used by the verification code explicitly. CI does not
  permit exceptions to this rule. Format verification code with normal include
  sorting enabled.

## Tests

- Every code addition or change must add or update a focused standalone test in
  `test/unittest/*.test.cpp`, and Codex must compile and run the relevant unit
  tests before committing.
- Exception: when the user explicitly asks for a new
  `test/verify/*.test.cpp`, that verification test may replace the unit-test
  requirement for that change. Run or otherwise validate the requested
  verification test whenever the necessary judge or verifier is available.
- Cover normal cases, boundary cases, and assumptions that are essential to the
  algorithm. For optimized algorithms, compare against a simple implementation
  on small or randomized inputs when practical.
- Use deterministic random seeds in randomized tests.
- Do not claim that a test passed unless it was actually run. If a required
  tool or external service is unavailable, report exactly what remains
  unverified.

## Documentation

- Add or update the corresponding file under `docs/` for every new algorithm
  and every code change that affects documented behavior, constraints,
  complexity, or usage.
- Write documentation content in Japanese.
- Create new pages from `templates/documentation.md`. Keep the required `title`
  and `documentation_of` front matter and follow the heading structure in the
  template.
- Document the overview, API signature, behavior, constraints or
  preconditions, and time and space complexity. Include other sections from the
  template when applicable.
- Do not write a literal `|` inside inline math because the documentation
  renderer may interpret it as a Markdown table delimiter. Prefer a named
  variable, or escape vertical bars in absolute values, such as `\|x\|`.
- After changing Markdown documentation, run:

  ```console
  npm run docs:check
  ```

## Git Workflow

- Use a dedicated branch for each feature or coherent change. Before editing,
  confirm that the current branch is dedicated to the task; create one if it is
  not.
- Keep unrelated user changes intact and out of the commit.
- Review the diff and run all relevant formatting, tests, and documentation
  checks before committing.
- Commit the completed change. Use a concise commit message that describes the
  feature or change.
- Do not rewrite, amend, squash, rebase, or force-push existing history unless
  the user explicitly requests it.

## Completion Report

When handing the task back to the user, summarize the implementation and list
the checks that were actually run. Mention the branch and commit, and clearly
identify any checks that could not be completed.

---
> Source: [ei1333/library](https://github.com/ei1333/library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
