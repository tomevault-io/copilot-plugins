## spec-driven-codex

> - `bin/` – CLI entry point (`cli.js`) wiring the `init` command and locale option handling.

# Repository Guidelines

## Project Structure & Module Organization
- `bin/` – CLI entry point (`cli.js`) wiring the `init` command and locale option handling.
- `lib/` – Command implementations and utilities. Currently only `commands/init.js` plus helpers under `lib/utils/`.
- `templates/` – Locale-specific assets copied during initialization (`templates/en` and `templates/ja` for prompts and `.sdd` skeleton files).
- `test/` – Node test suites exercising the `init` workflow (`test/init.test.js`).
- Top-level docs (`README.md`, `README.ja.md`, `CONTRIBUTING.md`, `DEVELOPMENT.md`, `AGENTS.md`) describe usage and contributor expectations.

## Build, Test, and Development Commands
- `npm install` – Install dependencies defined in `package.json`.
- `npm run lint` – Run ESLint (configured via `eslint.config.js`) across sources and tests.
- `npm test` – Execute Node.js tests (`node --test`) to validate `init` behavior across locales and overwrite scenarios.
- `npm link` / `npm link spec-driven-codex` – Register the local CLI globally and consume it from a sample project while iterating.

## Coding Style & Naming Conventions
- JavaScript (ES2022) with CommonJS modules; keep code ASCII and prefer descriptive function names (e.g., `resolvePaths`).
- Follow ESLint’s recommended + `eslint-plugin-security` guidelines; avoid suppressions unless reasoning is documented.
- Prompts and templates are named `sdd-*.md` and stored under locale folders; keep filenames lower-kebab-case.
- Use two-space indentation in Markdown docs and JavaScript files (match existing style when editing).

## Testing Guidelines
- Tests live under `test/` and use the built-in `node:test` runner.
- Name test files with `.test.js`; each test should set up its own temp directories (see `createTempDir`) and assert filesystem side effects.
- Always run `npm test` before committing; add new suites when extending `init` behavior or templates.

## Commit & Pull Request Guidelines
- Commit messages follow a Conventional Commit flavor (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`). Keep commits atomic and reference issues where applicable.
- Pull requests should describe the change, the impact on templates/locales, and list verification steps (`npm run lint`, `npm test`, manual CLI smoke tests).
- Include screenshots or terminal transcripts when modifying user-facing prompts or CLI output.

## Locale Handling Tips
- Default assets are English; pass `--locale ja` to exercise Japanese content.
- When adding locales, mirror the directory structure under `templates/<locale>/` and update tests to assert the new language paths.

---
> Source: [tomada1114/spec-driven-codex](https://github.com/tomada1114/spec-driven-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
