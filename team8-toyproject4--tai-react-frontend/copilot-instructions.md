## tai-react-frontend

> - `src/main.tsx` bootstraps the Vite + React + TypeScript app; `App.tsx` wires up routing with `react-router-dom`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/main.tsx` bootstraps the Vite + React + TypeScript app; `App.tsx` wires up routing with `react-router-dom`.
- Feature code lives under `src/components`, with reusable UI primitives in `src/components/ui` and data fetchers in `src/api`.
- Shared visual assets and tokens reside in `src/styles`. Markdown references and onboarding notes live in `src/guidelines`.
- Static entry files (`index.html`, `index.css`) sit at the repo root. Use `.env.example` as the template for environment variables.

## Build, Test, and Development Commands
- `npm install` — install dependencies (run once after cloning or when package.json changes).
- `npm run dev` — launch the Vite dev server on localhost with hot module reloading.
- `npm run build` — generate a production bundle; run it before opening a PR to catch type or bundling errors early.
- Optional container workflow: `docker-compose up --build` starts the app with the provided Dockerfile and ports.

## Coding Style & Naming Conventions
- Use TypeScript throughout; prefer functional components and hooks over class components.
- Follow the existing two-space indentation and single-quote imports. Keep JSX props on new lines when they wrap.
- Name React components and files in `PascalCase` (`TrendingList.tsx`), utility modules in `camelCase`, and CSS files in `kebab-case`.
- When adding layout or theme changes, reuse utilities in `src/components/ui` and class helpers (`clsx`, `tailwind-merge`) before creating new styles.

## Testing Guidelines
- No automated tests ship with the repo yet. Adopt Vitest + React Testing Library for component tests placed alongside source files as `ComponentName.test.tsx`.
- Ensure new features include happy-path and edge-case coverage; target at least the lines touched by your change.
- Add an npm script (e.g., `"test": "vitest"`) when you introduce tests and document how to run them in your PR.

## Commit & Pull Request Guidelines
- Follow conventional commit prefixes (`feat:`, `fix:`, `refactor:`) as seen in recent history; keep subject lines under 72 characters.
- Each PR should describe the intent, list major changes, and reference related issues. Include before/after screenshots for UI updates and note any new environment variables.
- Verify `npm run build` succeeds locally before requesting review; mention the result in the PR checklist and flag any known follow-up work.

## Configuration & Security Tips
- Start by copying `.env.example` to `.env` and fill in only the required values; never commit secrets.
- When collaborating, document any new environment keys in `.env.example` and coordinate rotations if a key leaks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Team8-ToyProject4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Team8-ToyProject4)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
