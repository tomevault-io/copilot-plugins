## ruleport

> <!-- Auto-synced from .cursor/rules/ -->

<!-- Auto-synced from .cursor/rules/ -->
<!-- DO NOT EDIT - Edit .cursor/rules/*/RULE.md instead -->

# AI Assistant Rules

This file consolidates all rules from cursor configuration.

---

# Always Apply Rules

These rules apply to all files:

## logging-standards

> Logging standards and best practices

# Logging Standards

## Logging Format
- Include timestamp, log level, message, and any relevant metadata.
- Use **log levels** (e.g., `info`, `warn`, `debug`, `error`).

## Error Logging
- Always include error messages and stack traces in error logs.

## Debug Logging
- Debug logs should be gated behind a `DEBUG` environment variable.
- Use **debug** level for verbose logging.

## Logging levels
- Use **info** level for general information.
- Use **warn** level for warnings.
- Use **debug** level for detailed debugging.
- Use **error** level for errors.

---

## project-standards

> Global project standards and best practices

# Project Standards

## Primary Reference Documents

**ALWAYS** refer to:
- `specifications/SPEC_CONSTITUTION.md` - Project-wide standards

## Tasks
Predefined tasks can be found in the ./specifications/tasks dir. Use these when crafting features and planning your work. You will be told which task to pick up.

## Commit Protocol
- Use **Conventional Commits** format: `type(scope): subject`.
  - Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.
  - Example: `feat(sync): add support for --target flag`

## Documentation
- **Keep README Updated**: Any change to a user's interaction with `ruleport` must be reflected in `README.md`.
- **Self-Documenting Code**: Complex logic in sync scripts must have comments explaining the "why", not just the "how".

## File Organization
- New features should be modular.
- Do not create files in the root directory unless they are critical configuration or entry points.

---

## typescript-standards

> Typescript coding standards

# TypeScript Standards

## Syntax
- Use **ES Modules** (`import` / `export`) for all TypeScript files.
- Use `const` for variables that don't change, `let` for those that do. Avoid `var`.
- Use **String Interpolation** (Template Literals) over concatenation.
- Use **Interfaces** for object definitions and public contracts.
- Use **Explicit Return Types** for public functions to ensure stable contracts.

## Async Patterns
- Prefer `async/await` over raw `.then()` chains.
- Always wrap top-level await implementations or main execution logic in try/catch blocks.

## Error Handling
- Scripts must exit with `process.exit(1)` on fatal errors.
- console.error messages should be prefixed with an emoji (e.g., ❌) for visibility in terminal logs.

## Dependencies
- Minimize external dependencies. Usage of native `fs`, `path`, and `child_process` is preferred.
- `yaml` is the only strictly required 3rd party dependency for parsing.
- Use `vitest` for testing.

---

## workflow-standards

> Workflow and usage guidelines for the sync tool

# Workflow Standards

## Dogfooding
- This project uses itself to manage its rules.
- **Source of Truth**: `.cursor/rules/`.
- **Derived Files**: `.agent/`, `.claude/`, `.github/`.
- **NEVER** edit derived files manually. Always edit the Source of Truth and run sync.

## Sync Protocol
- After editing any file in `.cursor/rules/`, you MUST run:
  ```bash
  npm run sync
  ```
- Before committing, ensure the sync output is clean and updated.

## Testing
- When modifying the sync core (`sync-rules-advanced.js`), verify with generic arguments:
  ```bash
  npm run sync -- --target copilot
  ```

---

# Conditional Rules

These rules apply to specific file patterns:

## javascript-standards

> Node.js and JavaScript coding standards

**Applies to:** `**/*.js`, `**/*.mjs`

# JavaScript Standards

## Syntax
- Use **CommonJS** (`require`) for this project to maintain compatibility with standard Node.js environments without build steps.
- Use `const` for variables that don't change, `let` for those that do. Avoid `var`.
- Use **String Interpolation** (Template Literals) over concatenation.

## Async Patterns
- Prefer `async/await` over raw `.then()` chains.
- Always wrap top-level await implementations or main execution logic in try/catch blocks.

## Error Handling
- Scripts must exit with `process.exit(1)` on fatal errors.
- console.error messages should be prefixed with an emoji (e.g., ❌) for visibility in terminal logs.

## Dependencies
- Minimize external dependencies. Usage of native `fs`, `path`, and `child_process` is preferred.
- `yaml` is the only strictly required 3rd party dependency.

---

---
> Source: [lewiesnyder/RulePort](https://github.com/lewiesnyder/RulePort) — distributed by [TomeVault](https://tomevault.io/claim/lewiesnyder).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
