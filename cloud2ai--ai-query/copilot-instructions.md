## ai-query

> Most active work lives in `hyperbdr-docs/`, the VuePress 2 documentation site for HyperBDR. Markdown content is under `hyperbdr-docs/src/`, with localized trees such as `src/zh/` and `src/ja/`, plus product-focused docs like `src/product-overview/` and `src/userguide/`. Site configuration, navbar, sidebar, theme, and styles live in `src/.vuepress/`. Static assets belong in `src/.vuepress/public/` or a section-local `images/` folder. Utility scripts are in `hyperbdr-docs/tools/`. Do not edit generated files in `src/.vuepress/dist/`.

# Repository Guidelines

## Project Structure & Module Organization
Most active work lives in `hyperbdr-docs/`, the VuePress 2 documentation site for HyperBDR. Markdown content is under `hyperbdr-docs/src/`, with localized trees such as `src/zh/` and `src/ja/`, plus product-focused docs like `src/product-overview/` and `src/userguide/`. Site configuration, navbar, sidebar, theme, and styles live in `src/.vuepress/`. Static assets belong in `src/.vuepress/public/` or a section-local `images/` folder. Utility scripts are in `hyperbdr-docs/tools/`. Do not edit generated files in `src/.vuepress/dist/`.

## Build, Test, and Development Commands
Run commands from `hyperbdr-docs/`:
- `yarn` installs dependencies.
- `yarn docs:dev` starts the local docs server.
- `yarn docs:clean-dev` starts the dev server with a cleared cache when content or config looks stale.
- `yarn docs:build` performs a production build.
- `node tools/docs-manager.cjs --build` processes external docs before publishing.
- `yarn docs:update-package` refreshes VuePress dependency metadata.

## Coding Style & Naming Conventions
Use lowercase, hyphen-separated filenames for Markdown, for example `dr-operations-manual.md`. Name images after the document plus a sequence number, for example `dr-operations-manual-1.png`. Keep headings concise and use existing VuePress config patterns in TypeScript files under `src/.vuepress/`. When adding a page, update the matching sidebar and navbar entries so it is reachable.

## Testing Guidelines
This repository does not appear to have a formal unit-test suite. Validate changes with `yarn docs:build`, then spot-check the rendered pages in `yarn docs:dev`. For doc imports or image updates, confirm the generated Markdown and linked assets render correctly in the browser.

## Commit & Pull Request Guidelines
Recent history uses short, imperative commit subjects such as `update ...`, `add ...`, and `fix ...`, usually lowercase. Keep commits focused on one doc set or tooling change. PRs should describe the content changed, list affected locale paths, and include screenshots or preview links when the rendered site changes. Mention any navbar, sidebar, or asset updates explicitly.

## Agent-Specific Notes
Prefer editing source Markdown and VuePress config, not generated output. If you touch localized content, check both the source locale and any mirrored navigation files before finishing.

---
> Source: [cloud2ai/ai-query](https://github.com/cloud2ai/ai-query) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
