## svetisite

> The app is a Next.js 15 App Router project. Core UI and routing live in `src/app`, with `layout.tsx` orchestrating shared HTML and `page.tsx` providing the landing page. Shared components belong under `src/app/components`, and translation resources live in `src/locales` with locale keys consumed through `providers.tsx`. Static assets (icons, logos) should be placed in `public/`; reference them via `/asset.svg`. Keep new modules TypeScript-first and respect the `@/*` path alias defined in `tsconfig.json`.

# Repository Guidelines

## Project Structure & Module Organization
The app is a Next.js 15 App Router project. Core UI and routing live in `src/app`, with `layout.tsx` orchestrating shared HTML and `page.tsx` providing the landing page. Shared components belong under `src/app/components`, and translation resources live in `src/locales` with locale keys consumed through `providers.tsx`. Static assets (icons, logos) should be placed in `public/`; reference them via `/asset.svg`. Keep new modules TypeScript-first and respect the `@/*` path alias defined in `tsconfig.json`.

## Build, Test, and Development Commands
Use `npm install` once per clone to hydrate dependencies. `npm run dev` launches the Turbopack dev server at `localhost:3000`; restart if you change eslint or TypeScript config. `npm run build` performs a production bundling pass and will surface type errors. `npm run start` serves the compiled output, and `npm run lint` runs ESLint with the `next/core-web-vitals` ruleset.

## Coding Style & Naming Conventions
Follow the ESLint guidance before committing; resolve all `npm run lint` warnings. Component files use the `.tsx` extension, PascalCase names, and two-space indentation (as in `src/app/header.tsx`). Hooks use camelCase (`useTranslation`), and translation keys stay snake-case to match the existing JSON. Prefer Tailwind utility classes for styling and colocate small helper modules alongside their consumers unless shared across routes.

## Testing Guidelines
A test runner is not wired up yet—set expectations when introducing one (Jest + React Testing Library or Vitest are both compatible). Place unit tests next to the module (`module.test.tsx`) or in `src/__tests__` if multiple suites share fixtures. Keep tests isolated from locale data; mock translation hooks instead of loading real JSON. Block merges on green tests once the script is in `package.json`.

## Commit & Pull Request Guidelines
Use imperative, <=72 character subject lines (e.g., `Add header language toggle`). Group related changes into a single commit; avoid mixing refactors with feature work. PRs should explain the user-facing impact, list testing performed, and attach screenshots for UI updates. Link to the relevant issue or ticket and call out any follow-up tasks.

## Localization Notes
When adding text, define keys in both `src/locales/german.json` and `src/locales/russian.json`. Keep keys grouped by feature (e.g., `nav.*`) and ensure new components pull translations through `useTranslation` so the toggle continues to work.

---
> Source: [maxf98/svetisite](https://github.com/maxf98/svetisite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
