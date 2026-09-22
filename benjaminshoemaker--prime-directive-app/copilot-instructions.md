## prime-directive-app

> - `src/app/(public)/` — App Router entrypoints: landing, Figma-aligned intake (`intake/[step]/`), loading, results, privacy.

# Repository Guidelines

## Project Structure & Module Organization
- `src/app/(public)/` — App Router entrypoints: landing, Figma-aligned intake (`intake/[step]/`), loading, results, privacy.
- `src/components/` — Tailwind primitives and overlays (buttons, `StepDetailOverlay`, accordion, modal shell).
- `src/lib/decision/` — Step registry, feature flags, and `buildPlan`.
- `src/lib/state/` — Zustand store with session storage + intake metadata and slider state.
- `src/lib/utils/` — Shared helpers (`cn`).
- `assets/`, `src/figma/` — Figma export for reference; excluded from lint/typecheck.
- Tests colocated with sources (e.g., `src/lib/state/store.test.ts`, `src/app/(public)/results/ResultsClient.test.tsx`).

## Build, Test, and Development Commands
- `npm run dev` — Start Next.js dev server at `http://localhost:3000`.
- `npm run build` / `npm run start` — Production build and launch.
- `npm run lint` — ESLint flat config (`eslint.config.mjs`); fails on warnings treated as errors.
- `npm run typecheck` — TypeScript `--noEmit` using `tsconfig.json`.
- `npm run test` — Jest + React Testing Library suite with `ts-jest`.

## Coding Style & Naming Conventions
- TypeScript + React with strict mode; prefer explicit types.
- TailwindCSS using tokens from `tailwind.config.ts`; keep class stacks aligned with Figma export.
- Components/modules PascalCase; hooks/selectors camelCase.
- Run `npm run lint` prior to commits (Figma export under `src/figma/**` is ignored).
- Follow existing folder segmentation for new UI/state modules.

## Testing Guidelines
- Frameworks: Jest + React Testing Library; Zustand store reducer tests.
- Place `.test.ts`/`.test.tsx` alongside implementation.
- Mirror existing coverage expectations (`store.test.ts`, `ResultsClient.test.tsx`, `StepDetailOverlay.test.tsx`).
- Run `npm run test -- --coverage` when significant UI/state changes land; avoid snapshots unless justified.

## Commit & Pull Request Guidelines
- Commit messages: imperative summary (~50 chars) followed by optional body (e.g., `feat: add roadmap modal`).
- Keep commits scoped; avoid bundling unrelated refactors.
- Pull requests should include: summary of changes, testing notes (`npm run lint`, `npm run test`), relevant screenshots/GIFs for UI work, and linked issue IDs if applicable.
- Verify `npm run lint`, `npm run typecheck`, and `npm run test` locally before requesting review.

## UI & Interaction Notes
- Learn More launches the full-screen `StepDetailOverlay`; keep copy/structure synchronized with Figma definitions.
- Intake slider state persists via `monthlyIncomeEstimate` in Zustand; update store tests when adding new actions.
- Confetti events (`canvas-confetti`) are mocked in tests; add similar mocks if firing animations elsewhere.

## Dependencies & Animations
- Motion uses `framer-motion`; reusable variants live inline per component.
- Confetti lives in UI client modules; import only inside client components to avoid SSR issues.
- Keep new dependencies declared in `package.json` and typed via `@types/*` when available.

---
> Source: [benjaminshoemaker/prime_directive_app](https://github.com/benjaminshoemaker/prime_directive_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
