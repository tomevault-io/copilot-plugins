## gmsv-web

> Guidance for AI agents (Devin, Claude, etc.) working in this repo.

# AGENTS.md

Guidance for AI agents (Devin, Claude, etc.) working in this repo.

## Repository layout

This is a monorepo with two independent npm workspaces (no root package.json):

- `studio/` — Sanity Studio (content management UI). React 19 + Sanity v6.
  - Schema types live in `studio/schemaTypes/*.ts` and are aggregated in
    `studio/schemaTypes/index.ts`.
  - `studio/sanity.config.ts` registers plugins and schema types.
  - `studio/sanity.cli.ts` holds the projectId/dataset and the typegen config.
    Typegen output is written to `web/sanity.types.ts` and the type path covers
    `web/src/**/*.{ts,tsx,js,jsx}`.
- `web/` — Public marketing/site frontend. React 19 + Vite 8 + Tailwind v4 +
  shadcn (base-nova style, `@` alias → `web/src`). Reads content from Sanity
  via `@sanity/client` (see `web/src/sanity/client.ts`).

Sanity project: `aqo7zrnm`, dataset: `production`.

## Environment

- Node 22 (tested on v22.21.1). Use nvm to install it — see README.md.
- `studio/.env` may hold a `SANITY_API_TOKEN` for scripted access. It is
  git-ignored; never commit secrets.
- `web/.env` must define `VITE_SANITY_PROJECT_ID` and `VITE_SANITY_DATASET`.

## Common commands

Run from inside the relevant workspace directory (`cd studio` or `cd web`).

### Studio (`cd studio`)
- `npm run dev` — start Studio dev server (http://localhost:3333).
- `npm run build` — build Studio to `dist/`.
- `npm run deploy` — deploy Studio to Sanity Cloud.
- `npm run deploy-graphql` — deploy the GraphQL API.
- `npm run typegen` — extract schemas and regenerate `web/sanity.types.ts`.

### Web (`cd web`)
- `npm run dev` — start Vite dev server.
- `npm run build` — typecheck (`tsc -b`) and build.
- `npm run lint` — eslint.
- `npm run preview` — preview the production build.

## Editing Sanity schemas

1. Add or modify a document/object type in `studio/schemaTypes/`.
2. Export it from `studio/schemaTypes/index.ts` (the `schemaTypes` array is
   also registered in `studio/sanity.config.ts`).
3. Run `cd studio && npm run typegen` to regenerate `web/sanity.types.ts`.
4. Update GROQ queries in `web/src/sanity/queries.ts` and any consumer
   components if the shape changed.
5. Run `cd studio && npm run deploy` to publish the updated Studio.

## Conventions

- Prettier (studio): `semi: false`, `singleQuote: true`, `printWidth: 100`,
  `bracketSpacing: false`. Match existing style when editing.
- Frontend uses the `@/` alias for `src/` imports.
- shadcn components live in `web/src/components/ui`. Use the shadcn skill when
  adding or modifying them.
- Use the sanity-best-practices skill for schema design, GROQ, and typegen
  questions; use the sanity-migration skill for CMS migration work.

## Verification

Before considering work done:
- For web changes: `cd web && npm run lint && npm run build`.
- For studio changes: `cd studio && npm run build` (and `npm run typegen` if
  schemas changed, then rebuild web to confirm types still satisfy).

---
> Source: [kku1993/gmsv-web](https://github.com/kku1993/gmsv-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
