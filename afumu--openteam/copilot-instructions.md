## openteam

> OpenTeam is a Manifest V3 Chrome extension built with Vite and TypeScript. Core source lives in `src/`: `background/` contains the service worker, routing, runtime, and control handlers; `content/` contains AI-site content scripts and adapters; `group/` contains group state, prompts, roles, and mention parsing; `teamPage/` contains the workspace UI. Shared utilities are in `src/shared/`, and protocol types are in `src/control/`.

# Repository Guidelines

## Project Structure & Module Organization

OpenTeam is a Manifest V3 Chrome extension built with Vite and TypeScript. Core source lives in `src/`: `background/` contains the service worker, routing, runtime, and control handlers; `content/` contains AI-site content scripts and adapters; `group/` contains group state, prompts, roles, and mention parsing; `teamPage/` contains the workspace UI. Shared utilities are in `src/shared/`, and protocol types are in `src/control/`.

Static extension files are in `public/`. Design docs and images are in `docs/`. The local agent CLI package is in `packages/openteamcli/`. `dist/` is generated build output; do not edit it directly.

## Build, Test, and Development Commands

- `npm install`: install dependencies.
- `npm run dev`: watch-build the extension in development mode.
- `npm run build`: create a production extension build in `dist/`.
- `npm run typecheck`: run strict TypeScript checks.
- `npm test`: run Vitest unit tests across `src/` and package tests.
- `npm run verify`: run typecheck, unit tests, and build before a PR.
- `npm run openteamcli -- doctor`: run the local CLI.

## Coding Style & Naming Conventions

Use TypeScript ES modules, 2-space indentation, single quotes, and no semicolons, matching existing files. Prefer explicit exported interfaces/types for shared contracts. Keep filenames camelCase for source and tests, with tests named beside code as `*.test.ts`. The project has no separate lint or formatting script; `tsconfig.json` enforces `strict`, `noUnusedLocals`, and `noUnusedParameters`.

## Testing Guidelines

Vitest is the test framework. Add focused unit tests beside changed modules, especially for routing, prompt construction, storage, UI state, and site adapters. Run `npm run verify` before changes that affect runtime behavior or packaging.

## Commit & Pull Request Guidelines

Recent history uses short imperative subjects, sometimes with Conventional Commit prefixes such as `feat:` or `docs:`. Keep commits scoped and descriptive, for example `feat: add group chat template` or `Fix duplicate orchestration retry handling`.

PRs should include a clear summary, test results, linked issues when relevant, and screenshots or recordings for UI changes. Call out permission, manifest, DNR rule, or AI-site adapter changes explicitly.

## Security & Configuration Tips

Review `public/manifest.json` and `public/openteam-frame-rules.json` carefully when changing permissions or frame behavior. Do not commit local browser data, generated `test-artifacts/`, or secrets.

---
> Source: [afumu/openteam](https://github.com/afumu/openteam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
