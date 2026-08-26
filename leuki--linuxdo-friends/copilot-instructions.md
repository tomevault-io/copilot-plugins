## linuxdo-friends

> This repository is a Chrome MV3 extension for linux.do. Source code lives in `src/`: `app/` contains the shared React UI, `side-panel/` and `options/` are extension entry points, `content/` injects linux.do page integrations, `background/` owns service-worker commands, `api/` and `domain/` hold parsing and business logic, and `state/` contains Jotai atoms. Static extension metadata is in `public/manifest.json`. Build output goes to `dist/`, release zips to `packages/`, and both are ignored. Project notes and planning artifacts live under `docs/` and `.omx/`.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Chrome MV3 extension for linux.do. Source code lives in `src/`: `app/` contains the shared React UI, `side-panel/` and `options/` are extension entry points, `content/` injects linux.do page integrations, `background/` owns service-worker commands, `api/` and `domain/` hold parsing and business logic, and `state/` contains Jotai atoms. Static extension metadata is in `public/manifest.json`. Build output goes to `dist/`, release zips to `packages/`, and both are ignored. Project notes and planning artifacts live under `docs/` and `.omx/`.

## Build, Test, and Development Commands

- `npm install`: install dependencies; use Node.js 22.
- `npm run dev`: watch-build the extension into `dist/` for local loading.
- `npm run build`: run TypeScript checking plus Vite builds for app and content script.
- `npm test`: run the Vitest suite.
- `npm run typecheck`: run `tsc --noEmit` only.
- `npm run set-version -- 1.2.3`: sync versions in `package.json`, `package-lock.json`, and `public/manifest.json`.
- `npm run package-extension -- --name linuxdo-friends-v1.2.3.zip`: package the built extension.

## Coding Style & Naming Conventions

Use TypeScript, React, Jotai, and existing module boundaries. Keep code ASCII unless a file already uses Chinese UI copy. Prefer small pure helpers in `domain/` or `api/`, and keep Chrome runtime/storage effects in `background/` or storage modules. Tests use `*.test.ts` or `*.test.tsx` beside the code they verify. Follow existing formatting: two-space indentation, semicolons, named exports, and concise CSS variable names in `src/styles/app.css`.

## Testing Guidelines

Use Vitest with jsdom where UI or content-script behavior is involved. Add focused tests for command contracts, parsing, state transitions, and content-script DOM integration. Before claiming a change is complete, run `npm test`, `npm run typecheck`, `npm run build`, and `git diff --check`. For content-script changes, also verify the built `dist/content-script.js` remains self-contained with no top-level import/export remnants.

## Security & Product Boundaries

Do not bypass Cloudflare, export or replay cookies, scrape outside normal logged-in browser behavior, or add a remote proxy/server for linux.do requests. Keep data local-first in extension storage and preserve explicit user-triggered refresh behavior unless a task explicitly changes it.

## Commit & Pull Request Guidelines

History currently uses simple messages such as `initial commit` and `release v1.2.0`. Keep commits focused and imperative. For PRs, include a short behavior summary, tests run, screenshots for UI changes, and any linux.do/browser-session assumptions.

---
> Source: [LeUKi/linuxdo-friends](https://github.com/LeUKi/linuxdo-friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
