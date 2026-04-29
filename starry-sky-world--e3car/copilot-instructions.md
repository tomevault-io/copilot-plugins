## e3car

> - Next.js App Router lives in `app/`; `app/page.tsx` is the redemption UI, `app/admin` holds the admin panel, and `app/api/...` contains route handlers for OAuth, Graph, and invite flows.

# Repository Guidelines

## Project Structure & Module Organization
- Next.js App Router lives in `app/`; `app/page.tsx` is the redemption UI, `app/admin` holds the admin panel, and `app/api/...` contains route handlers for OAuth, Graph, and invite flows.
- Shared logic sits in `lib/` (`oauth.ts`, `graph.ts`, `invite.ts`, `admin.ts`, `prisma.ts`).
- Styling is centralized in `app/globals.css` with Tailwind config in `tailwind.config.ts`.
- Database schema and migrations live in `prisma/` (`schema.prisma`, `migrations/`, local SQLite `dev.db`).
- Root configs: `next.config.mjs`, `tsconfig.json`, `postcss.config.js`, and ESLint via `next lint`.

## Setup, Build, Test, and Development Commands
- `npm install` — install dependencies.
- Create `.env` with Azure/Graph settings (`ENTRA_TENANT_ID`, `ENTRA_TENANT_DOMAIN`, `OAUTH_REDIRECT_URI`, `ADMIN_PASSWORD`, `DATABASE_URL=file:./dev.db`).
- `npx prisma migrate dev --name <tag>` — apply schema changes and generate the local SQLite database.
- `npm run prisma:generate` — regenerate Prisma client after schema edits.
- `npm run dev` — start the Next.js dev server on port 3000.
- `npm run build` then `npm start` — production build and serve.
- `npm run lint` — run ESLint/Next checks; keep the workspace clean before pushing.

## Coding Style & Naming Conventions
- TypeScript-first with 2-space indentation; avoid any file-level `any`.
- Components in PascalCase, functions/vars in camelCase, env keys in SCREAMING_SNAKE_CASE.
- Keep server-only code (tokens, Prisma client) in `app/api` or `lib` and do not import it into client components.
- Name Prisma migrations descriptively (`add-invite-expiry`) and keep schema formatting consistent with `prisma format` (invoked by migrate).

## Testing Guidelines
- No automated tests yet; rely on `npm run lint` plus manual verification of `/` and `/admin` flows.
- After schema changes, run `npx prisma migrate dev` and re-check both redemption and admin authorization paths.
- If adding tests, mirror the app router structure and name files `*.test.ts` near the code or in a `__tests__` folder.

## Commit & Pull Request Guidelines
- History favors short, imperative subjects (e.g., “Align Prisma versions…”); keep commits focused and scoped to a single concern.
- For PRs, include: purpose and linked issue, env/config changes, migration notes, and manual testing steps.
- Attach screenshots/GIFs for UI updates (admin and redemption screens) and mention any Graph/Azure prerequisites.

---
> Source: [Starry-Sky-World/E3Car](https://github.com/Starry-Sky-World/E3Car) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
