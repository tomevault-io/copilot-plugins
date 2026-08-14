## 250810-codex-cli-test

> - Root: `index.html` serves the static page. Keep it semantic and minimal.

# Repository Guidelines

## Project Structure & Module Organization

- Root: `index.html` serves the static page. Keep it semantic and minimal.
- Tooling: `.cursor/` stores editor/agent configs; do not ship production assets here.
- Suggested layout (when needed):
  - `styles/` for CSS (e.g., `styles/main.css`)
  - `scripts/` for JS modules (e.g., `scripts/app.js`)
  - `assets/` for images/fonts (e.g., `assets/logo.svg`)

## Build, Test, and Development Commands

- Local server: `python3 -m http.server 5173` then open `http://localhost:5173`.
- Alternative (Node): `npx http-server -p 5173` or `npx serve`.
- Build: none required (pure static). If you introduce a toolchain, document commands in this section.

## Coding Style & Naming Conventions

- HTML: 2-space indent, semantic tags (`<header>`, `<main>`, `<footer>`), lowercase attributes, quoted values.
- CSS: kebab-case class names; prefer BEM for components (e.g., `card`, `card__title`, `card--featured`). Define CSS variables in `:root`.
- JS (if added): ES modules in `scripts/`; camelCase for variables/functions, PascalCase for classes. Avoid globals; use `defer` on scripts.
- Formatting: Use Prettier. Example: `npx prettier --write .` (include HTML, CSS, JS).

## Testing Guidelines

- Current: no automated tests. Do quick checks before committing:
  - Page loads without console errors.
  - Basic a11y: focus visible, images with `alt`, landmarks present.
  - Validate HTML (e.g., W3C validator) and run `npx prettier --check .`.
- If adding tests, prefer Playwright for E2E (`tests/**/*.spec.ts`) and keep coverage for critical flows.

## Commit & Pull Request Guidelines

- Commits: follow Conventional Commits (e.g., `feat: add hero section`, `fix: correct nav contrast`). Keep messages imperative and scoped.
- Branches: `type/short-topic` (e.g., `feat/hero-layout`).
- PRs: clear description, linked issues, before/after screenshots for UI, and steps to verify locally. Keep diffs focused; update this file when workflows change.

## Security & Configuration Tips

- Prefer external JS/CSS files over inline; use `defer` and set `rel="preload"` when appropriate.
- Use relative paths for portability; avoid hard-coded origins.
- For third-party assets, use Subresource Integrity (SRI) and pin versions.

---
> Source: [dendedev/250810_codex-cli-test](https://github.com/dendedev/250810_codex-cli-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
