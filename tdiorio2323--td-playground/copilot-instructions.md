## td-playground

> Use this guide to ramp quickly on the Vite + React app.

# Repository Guidelines

Use this guide to ramp quickly on the Vite + React app.

## Project Structure & Module Organization
Source lives in `src/`: pages in `src/pages/`, shared UI in `src/components/`, hooks in `src/hooks/`, utilities in `src/lib/`, integrations and boot logic in `src/integrations/` + `src/boot/`. Entry points are `index.html`, `src/main.tsx`, and `src/App.tsx`. Assets stay in `public/`, previews in `previews/`, build and coverage output in `dist/` and `coverage/`. Co-locate tests beside modules or a sibling `__tests__/` and share setup via `src/setupTests.ts`. Design-library scripts sit under `tools/`.

## Build, Test, and Development Commands
- `pnpm install` installs dependencies with the pinned pnpm.
- `pnpm dev` runs Vite on http://localhost:8080 with HMR.
- `pnpm build` / `pnpm preview` generate and serve the production bundle.
- `pnpm build:dev` emits a readable bundle for debugging.
- `pnpm lint` runs ESLint across TS/TSX.
- `pnpm test` and `pnpm test:coverage` run Vitest locally or in CI.
- `pnpm routes:preview` (or `:fast`), `pnpm optimize:images`, and `pnpm lib:open|export|sync|version` refresh previews and maintain the design library.

## Coding Style & Naming Conventions
Write TypeScript React function components with 2-space indentation, semicolons, and the `@` path alias. Name components/pages in PascalCase, hooks in camelCase prefixed `use`, utilities as named exports, and constants as SCREAMING_SNAKE_CASE. Lean on Tailwind, shadcn/ui, Radix primitives, and the shared `cn()` helper. Let Prettier + ESLint autofix before commits.

## Testing Guidelines
Vitest and React Testing Library are wired via `vite.config.ts` and `src/setupTests.ts`. Name specs `*.test.ts(x)`, mock Supabase, router, and browser APIs for determinism, and colocate mocks with their subjects. Run `pnpm test:coverage` before merging auth, navigation, overlay, or command palette changes. Review snapshots rather than auto-accepting.

## Commit & Pull Request Guidelines
Use Conventional Commits (e.g., `feat: workspace filters`, `fix: auth guard redirect`) scoped to the area touched. Squash WIP branches before PRs. Each PR links the issue, summarizes behavior changes, adds screenshots or preview URLs for UI, notes QA steps and env vars, and captures manual migrations. Run `pnpm lint` and `pnpm test:coverage` locally and document any exceptions.

## Security & Configuration Tips
Keep secrets in `.env.local` with `VITE_` prefixes such as `VITE_SUPABASE_URL`, managing them via Supabase or Firebase dashboards. Validate new endpoints under `src/integrations/`, avoid checking in previews with real data, strip dev-only overlays before release, and run `pnpm optimize:images` on fresh media. Document role or ACL updates when adjusting Supabase config.

---
> Source: [tdiorio2323/Td-playground](https://github.com/tdiorio2323/Td-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
