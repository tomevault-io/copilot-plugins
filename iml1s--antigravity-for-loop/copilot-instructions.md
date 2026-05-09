## antigravity-for-loop

> - `extension.js` is the VS Code extension entrypoint; `plugin.json` is the legacy plugin manifest.

# Repository Guidelines

## Project Structure & Module Organization
- `extension.js` is the VS Code extension entrypoint; `plugin.json` is the legacy plugin manifest.
- `commands/` and `hooks/` contain bash scripts used by Antigravity; `workflows/` holds workflow definitions.
- `test/` houses JavaScript tests; `spec/` contains ShellSpec tests for scripts.
- `assets/` stores icons; `docs/` keeps design notes; `skills/` holds agent guidance; `reference_repos/` is for reference only.

## Build, Test, and Development Commands
- `npm install` installs dependencies.
- `npm test` runs shell and unit tests.
- `npm run test:shell` runs ShellSpec tests under `spec/`.
- `npm run test:unit` runs Mocha unit tests in `test/unit/`.
- `npm run test:integration` runs VS Code integration tests via `@vscode/test-cli`.
- `npm run test:e2e` runs VS Code E2E tests.
- `npm run package` builds a `.vsix` via `vsce package`.
- `npm run publish` publishes to Open VSX via `ovsx publish`.

## Coding Style & Naming Conventions
- JavaScript uses 4-space indentation, single quotes, and CommonJS `require`.
- Shell scripts target bash and should keep `set -euo pipefail`.
- Use kebab-case for script and workflow filenames (e.g., `for-loop.sh`, `for-loop.md`).
- Tests follow `*.test.js` for JavaScript and `*_spec.sh` for ShellSpec.

## Testing Guidelines
- Shell scripts are validated with ShellSpec (`spec/`).
- JavaScript tests use Mocha; E2E and integration use `@vscode/test-cli`.
- Add or update tests for behavior changes; no explicit coverage threshold is enforced.

## Commit & Pull Request Guidelines
- Recent history uses Conventional Commit style (`feat:`, `fix:`) and short imperative subjects; follow that where possible.
- PRs should describe behavior changes, include test commands run, and attach screenshots for status bar or Quick Pick UI changes.

## Configuration & Agent Notes
- Runtime state lives in `.antigravity/` and should never be committed (see `.gitignore`).
- `commands/` scripts assume `jq`; `git` is optional for branch creation.
- When changing loop behavior, update `workflows/for-loop.md`, `hooks/on_stop.sh`, and `skills/antigravity-for-loop/SKILL.md` together.

---
> Source: [ImL1s/antigravity_for_loop](https://github.com/ImL1s/antigravity_for_loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
