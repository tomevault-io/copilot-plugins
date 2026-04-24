## howbrowserswork

> This repository is an interactive guide to how browsers work, built with

# Repository Guidelines

## Purpose

This repository is an interactive guide to how browsers work, built with
Next.js and small interactive examples.

## Project Structure & Module Organization

-   `app/` contains the Next.js App Router entry points and global styles in
    `globals.css`.
-   `components/` holds reusable UI, examples, and section blocks:
    `components/examples/` for interactive demos and `components/sections/`
    for page sections.
-   `public/` is reserved for static assets.
-   Root configuration lives in `next.config.ts`, `tsconfig.json`,
    `eslint.config.mjs`, and `postcss.config.mjs`.

## Build, Test, and Development Commands

-   `npm run dev` starts the local Next.js development server.
-   `npm run build` produces the production build.
-   `npm run start` runs the production server after a build.
-   `npm run lint` runs ESLint checks.

## Coding Style & Naming Conventions

-   Stack: TypeScript + React (Next.js App Router) with Tailwind CSS.
-   Keep route segments and pages under `app/` with Next.js conventions
    (`page.tsx`, `layout.tsx`).
-   Name components in PascalCase and hooks/utilities in camelCase.
-   Formatting: rely on ESLint (`npm run lint`). There is no dedicated
    formatter, so mirror the existing file style.

## Testing Guidelines

-   No test framework is configured yet.
-   If you add tests, document the framework and add a `package.json` script
    to run them.

## Commit & Pull Request Guidelines

-   Git history is minimal and shows no enforced convention. Use short, imperative commit messages (e.g., "Add landing content").
-   PRs should include a brief purpose and change summary; include screenshots for UI changes.

## Configuration & Environment

-   Dependencies are managed via `package.json` and `package-lock.json`.
-   Run `npm run build` before `npm run start`.

---
> Source: [krasun/howbrowserswork](https://github.com/krasun/howbrowserswork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
