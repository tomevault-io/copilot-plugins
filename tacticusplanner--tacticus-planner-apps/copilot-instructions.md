## tacticus-planner-apps

> This repository is a pnpm/Turbo monorepo for a Vite React app with shared UI

# Repository Guidelines

## Project Structure & Module Organization

This repository is a pnpm/Turbo monorepo for a Vite React app with shared UI
and domain packages. New app code goes into Feature-Sliced Design (FSD) layers
under `apps/web/src/fsd` — see the `feature-sliced-design` skill before adding
files there.

- `apps/web`: the runnable React/Vite application. Source lives in
  `apps/web/src`, with `App.tsx`, `main.tsx`, the FSD tree (`src/fsd/`), and
  app-level config. Translations live in `apps/web/public/locales`.
- `packages/ui`: shared shadcn/Radix UI package exported as `@workspace/ui`.
  Components live in `packages/ui/src/components`, hooks in `src/hooks`,
  utilities in `src/lib`, and global Tailwind styles in `src/styles/globals.css`.
- `packages/game-catalog`: client for the server-side game catalog data
  (characters, MoWs, equipment, etc.) synced from `tacticus-planner-api`.
- `packages/game-domain`: shared game domain types/logic (rarity, ranks,
  progression rules) used across the app and `game-catalog`.
- `packages/player-data`: player-specific data models and persistence
  (IndexedDB/Dexie-backed storage).
- `.agents/skills` (canonical) / `.claude/skills` (generated mirror, kept in
  sync manually or via `skills-lock.json`): local agent skill docs — FSD,
  shadcn, naming conventions, i18n, tanstack-query, vitest, joyride tours,
  and `vulnerabilities-scan` (pnpm audit + fix process for this repo).
  Read the relevant skill before touching an area it covers. The V1→V2 page
  migration workflow (`reimplement-v1-page`) lives in the `tacticus-planner-dev`
  orchestration workspace instead, since it spans this repo, `tacticusplanner`,
  and `tacticus-planner-docs`.
- `.design-sync`: generated shadcn/ui component reference docs and previews
  under `.design-sync/docs` and `.design-sync/previews` — do not hand-edit;
  regenerated from the design system.
- Root files such as `turbo.json`, `pnpm-workspace.yaml`, `tsconfig.json`,
  `lefthook.yml`, and `.prettierrc` configure the workspace.

## Build, Test, and Development Commands

Use pnpm from the repository root.

- `pnpm install`: install workspace dependencies.
- `pnpm dev`: run Turbo development tasks, including the Vite dev server for `apps/web`.
- `pnpm dev:web`: run the web app in Aspire-integrated mode (used when the app
  is started from `tacticus-planner-api`'s Aspire AppHost; see that repo).
- `pnpm build` / `build:staging` / `build:production`: type-check and build
  packages/apps through Turbo for the given environment.
- `pnpm lint`: run ESLint + knip across configured workspace packages.
- `pnpm lint:fsd`: run `steiger` to check Feature-Sliced Design boundary rules
  under `apps/web/src/fsd`.
- `pnpm typecheck`: run TypeScript checks without emitting files.
- `pnpm format`: run Prettier formatting for TypeScript and TSX files.
- `pnpm test` / `pnpm test:run` / `pnpm test:coverage`: run the Vitest suite
  (watch / single-run / with coverage) across the workspace. Filter to one
  package with `pnpm --filter web test:run` or run a single file with
  `pnpm --filter web exec vitest run path/to/file.test.tsx`.
- `pnpm --filter web preview`: preview the built Vite app.
- `pnpm precommit`: run the lefthook pre-commit checks manually (format +
  lint on staged files).

## Coding Style & Naming Conventions

Use TypeScript and React function components. Follow the existing 2-space indentation, no semicolons, double quotes, LF line endings, and 80-column print width from `.prettierrc`. Prettier includes `prettier-plugin-tailwindcss`, so prefer letting formatting sort Tailwind classes.

Name React components in `PascalCase`, hooks as `useThing`, and utility files with short descriptive names such as `utils.ts`. Import shared UI through exports like `@workspace/ui/components/button` rather than reaching into package internals.

Data that crosses a boundary (network, IndexedDB, UI) follows the
Dto / StorageModel / domain / ViewModel naming tiers — see the
`naming-conventions` skill before naming a new boundary type or mapper.

## UI & Styling

This workspace uses shadcn/ui-style components, Radix primitives, lucide icons, and Tailwind CSS. Add shadcn components with:

```bash
pnpm dlx shadcn@latest add button -c apps/web
```

Generated shared components should land in `packages/ui/src/components`.
`.design-sync/docs` and `.design-sync/previews` hold generated reference docs
and previews for the design system — treat them as read-only output, not
something to hand-edit.

## Internationalization

User-facing strings go through `react-i18next`; translation resources live in
`apps/web/public/locales`. See the `react-i18next` skill for hook/namespace/
pluralization conventions before adding new strings.

## Testing Guidelines

Tests run on Vitest (`pnpm test`, `pnpm test:run`, `pnpm test:coverage`) and
are colocated with the code they cover, named `Component.test.tsx` or
`utils.test.ts` (see `apps/web/src/fsd/**/*.test.tsx` and
`packages/*/src/**/*.test.ts` for examples). See the `vitest` skill for
mocking, fixtures, and filtering conventions. Add new tests using the same
colocated pattern.

## Commit & Pull Request Guidelines

The current Git history uses short imperative subjects such as `Setup project`. Keep commit messages concise and action-oriented. A lefthook pre-commit hook formats and lints staged files automatically (`pnpm prepare` installs it after `pnpm install`).

Pull requests should include a brief summary, validation steps run (`pnpm lint`, `pnpm lint:fsd`, `pnpm typecheck`, `pnpm test:run`, `pnpm build`), linked issues when applicable, and screenshots or recordings for visible UI changes.

Open short-lived topic branches from `main` and squash-merge reviewed pull
requests back into `main`. Delete topic branches after merging. For supported
releases, fix defects on `main` first and backport the squash commit through a
pull request into `release/X.Y`. See `RELEASING.md` for the complete release
and patch process.

---
> Source: [TacticusPlanner/tacticus-planner-apps](https://github.com/TacticusPlanner/tacticus-planner-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
