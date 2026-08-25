## yashan-yinstall-bash

> This repository is currently an empty scaffold. Keep the top level reserved for

# Repository Guidelines

## Project Structure and Module Organization

This repository is currently an empty scaffold. Keep the top level reserved for
project configuration and documentation. As implementation begins, use a
predictable layout:

- `src/` for production source code, organized by feature or domain.
- `tests/` for automated tests that mirror the source layout.
- `assets/` for versioned static assets such as images, fixtures, or templates.
- `docs/` for design notes and contributor-facing documentation.

Avoid placing generated files, local caches, or build output under source
directories. Add their paths to `.gitignore` when tooling is introduced.

## Build, Test, and Development Commands

No build system, package manager, or test runner is configured yet. Do not
document or depend on assumed commands such as `npm test` until the relevant
tooling is committed. When adding a runtime, provide reproducible commands in
the root README and keep them aligned with automation. Typical commands should
cover local development, formatting, linting, testing, and production builds.

For example, a JavaScript project might expose `npm run dev`, `npm run lint`,
and `npm test` through `package.json`; a Python project might document its
virtual-environment setup and `pytest` invocation.

## Coding Style and Naming Conventions

Follow the formatter and linter adopted with the chosen language; do not mix
formatting styles within a module. Use 2 spaces for JSON, YAML, and JavaScript
or TypeScript unless the configured formatter says otherwise. Prefer clear,
domain-oriented names: `user-profile.ts`, `UserProfile`, and
`loadUserProfile`. Keep modules focused and avoid unexplained abbreviations.

## Testing Guidelines

Add tests with each behavior change. Place them under `tests/` or beside the
module only when that is the established framework convention. Name test files
after the unit under test, for example `user-profile.test.ts`. Cover normal
behavior, validation failures, and regressions; avoid reducing coverage when
refactoring.

## Commit and Pull Request Guidelines

There is no Git history yet, so no repository-specific commit convention can
be inferred. Use short imperative commit subjects, such as
`Add profile validation`, and keep unrelated changes separate. Pull requests
should explain the change, list verification performed, link relevant issues,
and include screenshots for visible UI changes.

## Security and Configuration

Never commit credentials, tokens, or machine-specific configuration. Commit a
sanitized example such as `.env.example` when environment variables are needed,
and document every required variable in the README.

---
> Source: [louis0755/yashan-yinstall-bash](https://github.com/louis0755/yashan-yinstall-bash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
