## md-wysiwyg-editor

> This is a VS Code extension for a WYSIWYG Markdown editor. Extension-host code lives in `src/`, including `extension.ts`, `MarkdownDocument.ts`, and utilities under `src/utils/`. Webview/browser code lives in `webview/`, with reusable UI modules in `webview/components/`, shared browser helpers in `webview/ui/`, and i18n helpers in `webview/i18n/`. Cross-boundary message types belong in `shared/`. Tests are colocated in `src/__tests__/` and `webview/__tests__/`; VS Code API mocks are in `__mocks__/`. Localized strings are in `package.nls*.json` and `l10n/`, while static extension assets are in `images/`.

# Repository Guidelines

## Project Structure & Module Organization

This is a VS Code extension for a WYSIWYG Markdown editor. Extension-host code lives in `src/`, including `extension.ts`, `MarkdownDocument.ts`, and utilities under `src/utils/`. Webview/browser code lives in `webview/`, with reusable UI modules in `webview/components/`, shared browser helpers in `webview/ui/`, and i18n helpers in `webview/i18n/`. Cross-boundary message types belong in `shared/`. Tests are colocated in `src/__tests__/` and `webview/__tests__/`; VS Code API mocks are in `__mocks__/`. Localized strings are in `package.nls*.json` and `l10n/`, while static extension assets are in `images/`.

## Build, Test, and Development Commands

Use pnpm, not npm or yarn.

- `pnpm install` installs dependencies using `pnpm-lock.yaml`.
- `pnpm build` bundles the extension and webview via `esbuild.mjs` into `dist/`.
- `pnpm watch` starts an incremental esbuild watcher for development.
- `pnpm test` runs all Vitest suites once.
- `pnpm test:watch` runs Vitest in watch mode.
- `pnpm test:coverage` generates V8 coverage reports.
- `pnpm package` creates a VSIX under `releases/`.

For manual testing, press F5 in VS Code to launch an Extension Development Host.

## Coding Style & Naming Conventions

Write strict TypeScript. Follow `.editorconfig`: UTF-8, LF, spaces, 4-space indentation, final newline, and trimmed trailing whitespace except in Markdown. Use camelCase for variables/functions and PascalCase for classes/providers. Keep webview-to-extension communication routed through `webview/messaging.ts`; update `shared/messages.ts` when payload contracts change. Webview CSS should use `--vscode-*` variables for theme compatibility.

## Testing Guidelines

Vitest is the test runner. Extension tests run in Node; webview tests run in jsdom with `webview/__tests__/setup.ts`. Name files `*.test.ts`, for example `src/__tests__/markdownDocument.test.ts` or `webview/__tests__/slug.test.ts`. Coverage targets `src/utils/**/*.ts`, `src/MarkdownDocument.ts`, and `webview/utils/**/*.ts`, with 70% line and function thresholds.

## Commit & Pull Request Guidelines

Git history uses English type prefixes, often with Chinese descriptions, for example `fix: 修复 workflow pnpm 版本冲突`, `chore: 版本升级到 0.1.0`, and `release: v0.1.6`. Prefer concise conventional prefixes such as `feat:`, `fix:`, `chore:`, `test:`, and `release:`. Create branches from `dev` when contributing. Pull requests should target `dev`, describe the user-facing change, list verification commands such as `pnpm build` and `pnpm test`, link issues when relevant, and include screenshots or GIFs for webview UI changes.

## Security & Configuration Tips

Do not commit generated VSIX files unless intentionally releasing. Avoid hard-coding image server credentials or local filesystem paths; use `markdownWysiwyg.*` settings declared in `package.json` for configurable behavior.

---
> Source: [git-xing/md-wysiwyg-editor](https://github.com/git-xing/md-wysiwyg-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
