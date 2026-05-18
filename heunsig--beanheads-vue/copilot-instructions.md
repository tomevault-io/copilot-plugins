## beanheads-vue

> Use when introducing functionality that didn’t previously exist.

# Git Commit Message Guidelines

1. Always write commit messages in **English**.

2. Follow the format below for commit messages:

   ```
   <type>(<scope>): <subject>

   <body>

   <footer>
   ```

   * `<subject>` (Required)

     * Write a concise, one-line summary of the change.
     * Use **imperative present tense** (e.g., use “add” instead of “added” or “adds”).
     * Do not capitalize the first letter.
     * Do not end with a period (`.`).
   * `<body>` (Optional)

     * Explain the reason for the change, technical details, decisions made, and other considerations.
     * Can span multiple lines. Make it understandable **why** this change was made and in what context.
     * If applicable, describe structural changes and their impact.
   * `<footer>` (Optional)

     * Include references to related issues, breaking changes, or review notes.
     * Examples:

       * `Refs:` or `Related to:` to reference issues
       * `BREAKING CHANGE:` to indicate critical changes

3. Follow the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/#summary).

## Conventional Commit Types

* **build**:
  Changes that affect the build system, tools, or external dependencies.
  Examples: Updates to `npm`, `yarn`, `pnpm`, or configuration changes in `webpack`, `vite`, etc.

* **ci**:
  Changes to Continuous Integration (CI) configuration or scripts.
  These do not affect application logic.
  Used for setting or modifying build/test/deploy processes with tools like GitHub Actions, Travis CI, Jenkins, etc.

* **docs**:
  Documentation-only changes.
  Examples: Updates to README files, API docs, developer guides, or inline comments.

* **feat**:
  Adds a **new feature** for the user or system.
  Use when introducing functionality that didn’t previously exist.

* **fix**:
  Fixes a **bug**.
  Use for resolving known issues or correcting unintended behavior.

* **perf**:
  Changes that improve **performance**.
  Examples: algorithm optimization, memory usage reduction, faster execution (without changing features).

* **refactor**:
  Code changes that neither fix bugs nor add features.
  Examples: renaming variables, extracting functions, removing duplication, reorganizing directory structure.

* **style**:
  Code formatting changes that don’t affect functionality.
  Examples: indentation, line breaks, whitespace, or semicolon changes.
  **Note**: This does not include UI changes like in CSS or HTML.

* **test**:
  Adds or updates **tests**.
  Includes unit tests, integration tests, mocks, test utilities, etc.
  Should not affect production logic.

* **chore**:
  Routine tasks that don't modify source or test files.
  Examples: dependency updates, `.gitignore` edits, renaming folders, or config changes unrelated to build/CI.

---
> Source: [Heunsig/beanheads-vue](https://github.com/Heunsig/beanheads-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
