## storylite

> Guide for AI coding agents working in the StoryLite monorepo.

# AGENTS.md

Guide for AI coding agents working in the StoryLite monorepo.

## What this repo is

StoryLite is a lightweight, Vite-powered alternative to Storybook for HTML, React, Svelte, Vue, and
Solid component stories. The repo is both the product source and the demo site.

Published npm packages live under `@storylite/*`. User-facing docs are in root `README.md`. Human
contributor docs are in `CONTRIBUTING.md`.

## Requirements

- Node.js 24+
- pnpm 11.2.2 (exact version in root `package.json`)

```sh
pnpm install
```

## Monorepo layout

| Path                    | Purpose                                                         |
| ----------------------- | --------------------------------------------------------------- |
| `packages/storylite`    | Core CLI, managed Svelte 5 manager app, public config/types API |
| `packages/contracts`    | Shared types and renderer contracts                             |
| `packages/preview-host` | Iframe bootstrap and live CSS mutation API                      |
| `packages/renderer-*`   | Optional framework adapters (react, svelte, vue, solid)         |
| `apps/web`              | Public demo site deployed to GitHub Pages                       |
| `apps/e2e`              | Playwright end-to-end tests for `apps/web`                      |
| `apps/storylite-html`   | Internal HTML story app used in development                     |
| `demos/*`               | Framework-specific demo projects                                |

Workspace filters:

```sh
pnpm -F @storylite/storylite <script>
pnpm -F @storylite/web <script>
pnpm -F @storylite/e2e <script>
```

## Architecture (where to edit what)

```
User project (.storylite/config.ts, *.stories.ts)
        |
        v
packages/storylite/bin/storylite.mjs   # CLI entry, Vite orchestration
        |
        +-- project-graph.mjs           # story discovery, renderer plugins
        +-- static-build.mjs            # static story pages
        +-- customization.mjs         # HTML hook injection
        |
        v
packages/storylite/src/                # Svelte 5 manager UI
        |
        +-- lib/renderers/runtime.ts    # story mounting in iframe/canvas
        +-- lib/storylite/*             # normalization, settings, toolbar
        |
        v
packages/preview-host                  # iframe document bootstrap
packages/contracts                     # StoryLiteStory, renderer adapter types
packages/renderer-*                    # framework-specific client/static renderers
```

### Change guidance

- CLI behavior, dev/build/preview flow: `packages/storylite/bin/`
- Manager UI, sidebar, controls, toolbar: `packages/storylite/src/lib/components/`
- Story format, args, controls, normalization: `packages/contracts` +
  `packages/storylite/src/lib/storylite/`
- Iframe bootstrapping / CSS injection: `packages/preview-host`
- Framework rendering: matching `packages/renderer-*`
- Demo site content and config: `apps/web/.storylite/` and `apps/web/src/`
- Browser regressions: `apps/e2e/tests/apps/web/`

## Common commands

Run from repo root unless noted.

```sh
pnpm dev                 # all workspace dev servers
pnpm dev:web             # apps/web only
pnpm dev:storylite       # core package dev server
pnpm build               # build all workspaces
pnpm build:packages      # packages only
pnpm build:web           # apps/web only
pnpm typecheck           # all workspaces
pnpm format              # write formatting
pnpm format:check        # CI formatting gate
pnpm test                # unit tests (Vitest)
pnpm test:e2e            # Playwright e2e
pnpm qa                  # typecheck + format + build + test + e2e
```

First-time e2e setup:

```sh
pnpm -F @storylite/e2e run e2e:install
```

Scoped examples:

```sh
pnpm -F @storylite/storylite run typecheck
pnpm -F @storylite/storylite run test
pnpm -F @storylite/web run build:pages
```

## Coding conventions

### General

- ESM only (`"type": "module"`).
- TypeScript everywhere except a few `.mjs` CLI helpers.
- Keep diffs focused. Match surrounding style and naming.
- Do not add comments for obvious code.
- Do not over-engineer helpers or defensive branches for unlikely cases.

### Formatting

Use `oxfmt` (root `.oxfmtrc.json`):

- no semicolons
- single quotes
- print width 100
- trailing commas
- `@storylite/*` imports sorted first

Run `pnpm format` before finishing. CI runs `pnpm format:check`.

### Package builds

- Renderer/contract packages build with `tsdown` to `dist/`.
- `@storylite/storylite` ships its source via `exports`; the CLI bundles at runtime with Vite.
- After changing `@storylite/contracts`, rebuild dependent packages before testing consumers.

### Svelte (core manager app)

The manager app uses **Svelte 5** runes (`$props`, `$state`, `$derived`).

When creating or editing `.svelte` files in `packages/storylite`, use the Svelte MCP server tools
for docs lookup and validation when available.

Typecheck the core package with:

```sh
pnpm -F @storylite/storylite run typecheck
```

### StoryLite product constraints

StoryLite is a focused CSF-like subset, not full Storybook. Do not add or assume:

- `play`, loaders, decorators, docs/autodocs, actions, addon APIs
- arbitrary project callbacks in toolbar tools
- automatic merge of the consuming project's full `vite.config.ts`

Supported story features: `title`, `args`, `argTypes`, `parameters`, `render`, renderer selection,
static output.

Config and renderer adapter changes often require restarting `storylite dev`.

HTML customization hooks are trusted project-source HTML only.

## Testing

### Unit tests (Vitest)

- Core logic: `packages/storylite/src/**/*.test.ts`, `*.test.mjs`
- Preview host: `packages/preview-host/src/index.test.ts`
- Smoke tests in apps/demos verify package wiring

Prefer testing behavior over snapshots. Add tests when fixing bugs or changing non-trivial logic.

### E2E (Playwright)

- Tests live in `apps/e2e/tests/apps/<app>/`
- Shared helpers in `apps/e2e/tests/shared/`
- Keep route lists, fixtures, and app-specific assertions inside the app folder
- Default target: built static `@storylite/web` on `http://127.0.0.1:6340`
- See `apps/e2e/README.md` for env vars

Do not commit `test-results/`, `playwright-report/`, traces, screenshots, or videos.

## CI expectations

PR workflow (`.github/workflows/pr-quality.yml`):

1. `pnpm format:check`
2. `pnpm build`
3. `pnpm typecheck`
4. `pnpm test`

GitHub Pages deploy (`.github/workflows/gh-pages.yml`, `main` only):

1. format, build packages, typecheck
2. `pnpm -F @storylite/web run build:pages`
3. test
4. deploy `apps/web/dist-storylite`

Before opening a PR, run at least:

```sh
pnpm typecheck && pnpm format:check && pnpm build && pnpm test
```

Run `pnpm qa` when touching manager UI, CLI, static build, or e2e-covered flows.

## Git and PRs

- Use Conventional Commits: `feat(scope):`, `fix(scope):`, `docs(scope):`, etc.
- Only commit or push when explicitly asked.
- Do not amend commits unless the user requests it and the commit has not been pushed.
- Do not skip hooks or force-push to `main`.

Example:

```txt
feat(renderer-react): add static story rendering
fix(cli): preserve configured base path in static output
docs(readme): update install instructions
```

## Agent workflow checklist

1. Identify the smallest workspace/package that owns the change.
2. Read nearby files before editing; follow existing patterns.
3. Make the minimal correct change.
4. Run scoped typecheck/tests first, then broader checks if needed.
5. Format with `pnpm format`.
6. Summarize what changed and which commands were run.

## Useful references

- User docs: `README.md`
- Contributor commands: `CONTRIBUTING.md`
- E2E layout/env: `apps/e2e/README.md`
- Public API/types: `packages/storylite/src/public.ts`, `packages/contracts/src/index.ts`
- CLI entry: `packages/storylite/bin/storylite.mjs`

---
> Source: [itsjavi/storylite](https://github.com/itsjavi/storylite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
