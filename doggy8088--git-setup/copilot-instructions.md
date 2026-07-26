## git-setup

> - `bin/index.js` is the CLI entrypoint that configures Git settings and aliases.

# Repository Guidelines

## Project Structure & Module Organization
- `bin/index.js` is the CLI entrypoint that configures Git settings and aliases.
- `package.json` defines the npm package metadata, entrypoint, and scripts.
- `README.md`, `CHANGELOG.md`, and `PUBLISH.md` document usage and releases.
- `.github/` contains repository automation (issues, workflows, or templates).
- Root dotfiles like `.gitattributes` and `.gitignore` define repo behavior.

## Build, Test, and Development Commands
- `npm install`: install dependencies (none at runtime, but keeps lockfile in sync).
- `npm run start`: run the CLI locally (`node bin/index.js`).
- `npx @willh/git-setup`: run the published package (useful for smoke checks).
- `npm run bump`: bump patch version and regenerate `package-lock.json`.

## Coding Style & Naming Conventions
- JavaScript (Node.js) with 4-space indentation and semicolons.
- Prefer `const` for bindings; use `let` only when reassignment is needed.
- Use camelCase for variables and functions (e.g., `buildAttributesAlias`).
- Strings generally use single quotes; template literals for multi-line blocks.
- No formatter or linter is configured; keep changes consistent with `bin/index.js`.

## Testing Guidelines
- No automated test framework is set up yet, and there is no `npm test` script.
- Manual checks: run `npm run start` and verify expected `git config --global` output.
- If adding tests, place them in `tests/` and wire a `test` script in `package.json`.

## Commit & Pull Request Guidelines
- Use Conventional Commits 1.0.0 (e.g., `feat: ...`, `fix: ...`, `chore: ...`).
- Commit messages are expected to be concise and may be Traditional Chinese, matching `git ac` behavior.
- Update `CHANGELOG.md` and `README.md` when user-facing behavior or aliases change.
- PRs should include a short summary, validation steps, and OS notes for platform-specific changes.

## Configuration & Safety Notes
- The CLI modifies global Git config (`~/.gitconfig`) and may set `LC_ALL`/`LANG` on Windows.
- When changing alias behavior, document any exclusions or prompts in `README.md`.

---
> Source: [doggy8088/git-setup](https://github.com/doggy8088/git-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
