## ailens

> - `src/` TypeScript sources for the VS Code extension (entry: `extension.ts`).

# Repository Guidelines

## Project Structure & Module Organization
- `src/` TypeScript sources for the VS Code extension (entry: `extension.ts`).
- `src/webview/` static assets for the dashboard webview copied to `out/` on build.
- `out/` compiled JavaScript (generated; do not edit).
- `.tests/` Jest tests, mocks for `vscode` and `sqlite3`.
- Key modules: analyzers (`ai-analyzer.ts`, `code-operation-analyzer.ts`, `timing-analyzer.ts`), storage/DB (`ai-code-storage.ts`, `database.ts`), UI (`dashboard-webview.ts`).

## Build, Test, and Development Commands
- `npm install` install dependencies.
- `npm run compile` build TypeScript and copy webview assets to `out/`.
- `npm run watch` rebuild on change.
- `npm run lint` run ESLint over `src/`.
- `npm test` run Jest (ts-jest) with repo mocks and produce coverage.
- `npm run package` build a `.vsix` via `vsce`.
- `npm run watcher` optional helper to watch Cursor DB events (dev aid).

## Coding Style & Naming Conventions
- Language: TypeScript (Node ≥16, VS Code engine ^1.74).
- Indentation: 2 spaces; prefer semicolons.
- ESLint: `@typescript-eslint` rules enabled; variables/classes must be `camelCase`/`PascalCase`.
- Filenames: `kebab-case.ts` (e.g., `document-monitor.ts`). Classes/interfaces: `PascalCase`. Functions/vars: `camelCase`.
- Place shared helpers in `src/utils/`. Do not import from `out/`.

## Testing Guidelines
- Framework: Jest with `ts-jest`. Tests live in `.tests/` and end with `.test.ts` (e.g., `.tests/database.test.ts`).
- Mocks: repo provides `vscode` and `sqlite3` mocks; tests should not hit real VS Code or local DBs.
- Commands: `npm test` or `npm test -- --coverage`. Aim for ≥80% line coverage on changed areas.

## Commit & Pull Request Guidelines
- Current history contains short WIP commits (`--wip-- [skip ci]`). For contributions, use Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`.
- PRs must include: clear description, linked issues, screenshots/GIFs for UI changes (dashboard), and notes on testing.
- Before opening a PR: run `npm run lint`, `npm test`, and `npm run compile`. Do not commit `out/` or `.vsix` artifacts.

## Security & Configuration Tips
- Never commit personal data from `~/.ailens/` or Cursor databases; tests must use mocks.
- User settings keys exposed by the extension: `ailens.autoStart`, `ailens.showNotifications`, `ailens.debug`.
- Keep file I/O and DB access defensive (read-only where possible) and behind typed interfaces in `src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaiye)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaiye)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
