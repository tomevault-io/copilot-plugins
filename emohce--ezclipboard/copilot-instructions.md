## ezclipboard

> - `src/` contains the Vue 3 app entry (`main.js`) and UI modules. Core areas: `views/` (pages), `cpns/` (components), `hooks/`, `utils/`, `style/`, `global/`, `data/`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` contains the Vue 3 app entry (`main.js`) and UI modules. Core areas: `views/` (pages), `cpns/` (components), `hooks/`, `utils/`, `style/`, `global/`, `data/`.
- `public/` hosts uTools plugin assets such as `plugin.json`, `preload.js`, `listener.js`, and `index.html`. These files are copied into `dist/` on build.
- `docs/` includes supplementary documentation. `dist/` is build output and should be treated as generated.

## Build, Test, and Development Commands
- `npm install` installs dependencies.
- `npm run serve` starts the Vue dev server (configured to `http://localhost:8081/` in `vue.config.js`).
- `npm run build` produces a production build into `dist/` with a relative `publicPath` for uTools packaging.

## Coding Style & Naming Conventions
- Use 2-space indentation and keep Vue SFCs (`.vue`) organized with `<template>`, `<script setup>`, and optional `<style>`.
- Prefer PascalCase for components (`Main.vue`, `Setting.vue`) and kebab-case for custom events (`@show-setting`).
- Use plain JavaScript (no TypeScript currently). No repo-level lint/format config is present, so align with existing file style.

## Testing Guidelines
- No automated test framework is configured yet.
- For changes, run `npm run serve` to validate UI flows and ensure the uTools plugin works by loading `dist/` after a build.
- If adding tests, document the framework and add a script in `package.json`.

## Commit & Pull Request Guidelines
- Commit history follows Conventional Commits, e.g. `feat(ng-admin): ...`, `refactor(ng-admin): ...`, `chore(docs): ...`.
- Include a concise summary line and, when needed, a short bullet list of notable changes.
- PRs should describe behavior changes, include UI screenshots when visuals change, and note any required updates to `public/plugin.json` (e.g., version bump) before packaging.

## Packaging Notes (uTools)
- After `npm run build`, ensure `dist/plugin.json` is present at the root of the bundle.
- Package `dist/` as a zip and rename to `.upx` per uTools conventions (example: `ClipboardManager-1.0.0.upx`).

---
> Source: [emohce/EzClipboard](https://github.com/emohce/EzClipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
