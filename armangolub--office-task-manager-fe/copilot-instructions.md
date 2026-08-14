## office-task-manager-fe

> Guidance for working in this repository (office-task-manager-fe).

# CLAUDE.md

Guidance for working in this repository (office-task-manager-fe).

## Project

Frontend SPA for an office task manager. Milestone **M2** — building the frontend
foundation (Epic F0) that all future screens (F1–F6) build on. Production context is a
WebView inside a mobile platform; for now we ship a standard SPA.

## Stack

| Area        | Choice                                                         |
| ----------- | -------------------------------------------------------------- |
| Build       | Vite 8 + React 19 + TypeScript 6                               |
| Styling     | Tailwind **v4** (CSS-first, `@tailwindcss/vite`) + shadcn/ui   |
| Data        | TanStack Query (server state) + Zustand (client state) — F0.4  |
| API codegen | orval → TanStack Query hooks over a hand-written axios mutator |
| Routing     | React Router — F0.5                                            |
| i18n        | react-i18next (ru/en) — F0.6                                   |
| Forms       | React Hook Form + Yup — F0.7                                   |
| Tests       | Vitest — F0.2                                                  |
| Pkg manager | **Yarn** (yarn.lock; do not introduce npm/pnpm lockfiles)      |

## Commands

```bash
yarn dev            # start dev server
yarn build          # tsc -b && vite build (type-check + production build)
yarn lint           # eslint
yarn format         # prettier --write .
yarn format:check   # prettier --check .
# yarn test         # vitest (added in F0.2)
# yarn gen:api      # orval — regenerate API hooks from openapi (added in F0.3)
```

## Git workflow

- **Work directly on `master`.** Do NOT create feature branches or open MRs/PRs — the project
  is pre-launch with a single developer.
- **Do NOT run `git commit` (or `git add`/commit-like actions). The developer commits
  manually.** Make and verify the changes, then leave them in the working tree for the
  developer to review and commit.

## Verification

- **Do NOT manually/visually test the result yourself** (no preview server, no browser MCP,
  no clicking through flows). Type-check and lint are fine (`yarn build`, `yarn lint`,
  `yarn format:check`), but the developer verifies behaviour in the running app.
- **At the end of a sprint/epic, produce a hand-off checklist**: a concrete, step-by-step
  list of what the developer should click through to verify each piece of functionality that
  was built. This replaces self-testing.

## Architecture — Feature-Sliced Design (FSD)

Layers, from highest to lowest. **Imports only point downward** (a layer may import from
layers below it, never above):

```
app → pages → widgets → features → entities → shared
```

- `app/` — entry, providers, router, global styles.
- `pages/` — route-level screens.
- `widgets/` — composite UI blocks (e.g. app shell / navigation).
- `features/` — user-facing capabilities (auth/login, logout…).
- `entities/` — business entities + their stores (e.g. `session`).
- `shared/` — framework-agnostic building blocks: `api/`, `config/`, `lib/`, `ui/`, `i18n/`.
  **`shared` must not import from any upper layer.**

### Key layering constraint (auth token)

`shared/api/http` (the axios mutator) needs the access token, but `shared` cannot import
from `entities`. Resolution: a low-level in-memory token holder lives in
`shared/api/auth/token-store.ts` (get/set/clear + an `onUnauthorized` callback registration).
The Zustand session store in `entities/session` mirrors the token into it and registers the
logout handler. See `docs/http-auth.md` (added in F0.2).

## Conventions

- **Comments: avoid them.** Write self-explanatory code (clear names, small functions)
  instead of comments. Add one only when it explains _why_ something non-obvious is done
  (a workaround, a spec quirk, a layering constraint) — never to restate _what_ the code
  does. No JSDoc/banner comments on ordinary functions and components.
- Path alias `@/*` → `src/*` (configured in `tsconfig.app.json`; Vite resolves it natively
  via `resolve.tsconfigPaths`).
- shadcn config in `components.json`: components → `@/shared/ui`, util → `@/shared/lib/cn`.
- Prettier: no semicolons, single quotes, trailing commas, width 80, Tailwind class sorting.
- The orval-generated folder `src/shared/api/generated/` is committed but excluded from lint
  and prettier (it's machine-generated).

## Docs

See `docs/`: `architecture.md`, `http-auth.md`, `auth-flow.md`, `api.md`, `i18n.md`,
`pages.md`, `design-foundation.md` (visual layer source of truth), `patterns.md` (reference
screen patterns to replicate across M2).

---
> Source: [ArmanGolub/office-task-manager-fe](https://github.com/ArmanGolub/office-task-manager-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
