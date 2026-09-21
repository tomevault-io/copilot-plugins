## mh-scanner

> The Next.js App Router lives in `app/`, with `(auth)` handling the passcode flow and `(dashboard)` hosting review screens such as `dashboard/episodes/[episodeId]`. Shareable UI sits in `components/` (forms, episodes, fragments) while backend orchestration is in `convex/` (schema, actions, crons). Domain services and helpers stay under `lib/` (`integrations`, `processing`, `types`), and automation scripts (Apify/OpenAI smoke checks) reside in `scripts/`. Keep new hooks in `hooks/` and static assets in `public/`.

# Repository Guidelines

## Project Structure & Module Organization
The Next.js App Router lives in `app/`, with `(auth)` handling the passcode flow and `(dashboard)` hosting review screens such as `dashboard/episodes/[episodeId]`. Shareable UI sits in `components/` (forms, episodes, fragments) while backend orchestration is in `convex/` (schema, actions, crons). Domain services and helpers stay under `lib/` (`integrations`, `processing`, `types`), and automation scripts (Apify/OpenAI smoke checks) reside in `scripts/`. Keep new hooks in `hooks/` and static assets in `public/`.

## Build, Test, and Development Commands
- `npm run dev` – start the local Next.js app and Convex client hot reload.
- `npm run build` – produce the production bundle; run before tagging releases.
- `npm run start` – serve the built bundle for smoke testing.
- `npm run lint` – enforce ESLint rules (`next/core-web-vitals`, `@typescript-eslint`).
- `npm run type-check` – run strict TypeScript without emitting output.
- `npm run format` – apply Prettier formatting; use prior to large diffs.
- `npm run convex:dev` / `npm run convex:deploy` – boot the Convex backend locally or deploy it.

## Coding Style & Naming Conventions
TypeScript is mandatory everywhere; prefer server components unless `'use client'` is required. Prettier enforces 2-space indentation, 80-column wrapping, semicolons, and double quotes; avoid manual formatting drift. Follow ESLint defaults, including `prefer-const`, and silence unused placeholders with a leading underscore. Folder names stay kebab-case, React components PascalCase, utility files camelCase.

## Testing Guidelines
We rely on static guarantees first: run `npm run lint` and `npm run type-check` before requesting review. Integration smoke tests live in `scripts/test-*.js`; execute them with `node scripts/test-openai.js` or similar when touching matching subsystems. Name ad-hoc test helpers with a `.spec-support.ts` suffix and keep them collocated.

## Commit & Pull Request Guidelines
Match the existing conventional style: `<type>: <imperative summary>` using tags like `fix`, `chore`, `docs`, or `feat`. Scope commits narrowly and note any config changes in the body. Pull requests should link the relevant Linear/Jira ticket, describe user-visible changes, list validation commands, and attach UI screenshots or Convex logs when behavior shifts.

## Security & Configuration Tips
Never hardcode secrets; mirror updates in `.env.local.example` and load runtime values via `npx convex env set NAME VALUE`. Confirm `YOUTUBE_API_KEY` and `APIFY_TOKEN` exist before running `npm run convex:dev`, and rotate tokens after demos. For OpenAI or Google credentials, store them only in Convex and avoid logging responses that could contain PII.

---
> Source: [joeoviedop/mh-scanner](https://github.com/joeoviedop/mh-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
