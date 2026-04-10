## inslab-website

> This repository uses Next.js App Router with source code under `src/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository uses Next.js App Router with source code under `src/`.

- `src/app/`: route entries and Next.js convention files such as `layout.tsx`, `page.tsx`, and route pages like `publications/page.tsx`
- `src/components/`: shared UI and layout components such as the site header and footer
- `src/features/`: domain-focused code grouped by feature (`home`, `publications`), including components, local data, and types
- `src/lib/`: shared utilities and motion presets
- `public/`: static assets served directly, including `public/images/`
- `docs/` and `resources/`: internal documentation and non-runtime design/source materials

Keep route files thin and move reusable UI into `src/components/` or `src/features/`.

## Build, Test, and Development Commands
Use `npm`; `package-lock.json` is committed.

- `npm install`: install dependencies
- `npm run dev`: start the local dev server at `http://localhost:3000`
- `npm run build`: create a production build and catch compile-time issues
- `npm run start`: run the production build locally
- `npm run lint`: run ESLint for the codebase

Before opening a PR, run `npm run lint` and `npm run build`.

## Coding Style & Naming Conventions
Write TypeScript with 2-space indentation and match nearby files.

- Use `PascalCase` for React components, `camelCase` for variables/functions, and lowercase route segment names under `src/app/`
- Prefer Server Components; add `"use client"` only when client-side interaction or motion requires it
- Use Tailwind utility classes inline in JSX; promote repeated values to global tokens in `src/app/globals.css`
- Use `next/link` for internal navigation and prefer `next/image` for images
- Avoid `any`; start with `unknown` if a type is unclear

## Testing Guidelines
There is no dedicated test framework configured yet. For now, treat these as required validation:

- `npm run lint`
- `npm run build`

When tests are added, place them near the code they cover or in a `tests/` directory. Prefer names like `page.test.tsx` or `home.spec.ts`.

## Commit & Pull Request Guidelines
Recent history follows Conventional Commit-style messages such as `feat(home): add stitched landing page and mega menu`.

- Follow the repo’s Git Flow workflow: branch feature work from `develop` into `feature/*`, use `release/*` for release prep, and `hotfix/*` from `main` for urgent production fixes
- Format commits as `type(scope): subject`
- Use clear scopes like `home`, `publications`, or `layout`
- Keep subjects short, imperative, and specific

PRs should follow `.github/PULL_REQUEST_TEMPLATE.md` and include a brief user-facing summary, linked issues when available, screenshots for UI changes, and the validation performed. If structure or workflow rules change, update the related docs in the same PR. Before requesting review, run `npm run lint`, complete relevant manual checks, and remove temporary files or debug code.

## Security & Configuration Tips
Do not commit secrets or machine-specific values. Store local configuration in `.env` files and document new runtime variables in `README.md` if they become required.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LSH9132)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LSH9132)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
