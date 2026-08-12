## pinmy

> Save links ("pins") by texting them to a Twilio number or via the web UI. Pins get scraped, chunked, AI-classified, and are full-text searchable. Users log in with phone number + OTP.

# pinmy

Save links ("pins") by texting them to a Twilio number or via the web UI. Pins get scraped, chunked, AI-classified, and are full-text searchable. Users log in with phone number + OTP.

## Layout (pnpm + turbo monorepo, Node 22)

- `apps/web` — Next.js 16 / React 19 / Tailwind 4, deployed on Vercel. Auth, UI, and the `/api/*` routes live here.
- `apps/api` — Hono server (`src/index.ts`, port 8080), deployed on Railway. Twilio inbound webhook + async pin processing.
- `packages/db` — `@pinmy/db`. Prisma 7, schema in `prisma/schema.prisma`, client generated into `src/generated` (never edit). Exports `prisma` singleton, `searchPins` (raw-SQL Postgres FTS over `tsv` columns), `decodeEntities`.
- `packages/queue` — `@pinmy/queue`. `MessageQueueClient.publish(endpoint, body)` = QStash publish to `BACKEND_API_URL + endpoint`.
- `packages/config` — `@pinmy/config`. `CATEGORIES` / `Category` / `CATEGORY_COLORS`, shared by web and api.

## Pin pipeline (the core flow)

1. Create: SMS -> `POST /webhook/twilio` (apps/api, form-encoded `From`/`Body`), or web -> `POST /api/pins` (apps/web, session auth). Both create a Pin with `status: "PROCESSING"` and publish `{phone, link, pinUniqueId}` to QStash.
2. Process: QStash calls back `POST /webhook/process` (apps/api): `scrapeLink` (regex-based HTML scrape, no DOM lib) -> `chunkText` -> `classifyPin` (Claude Haiku, falls back to `"Other"`) -> updates Pin + `PinChunk` rows.
   - Special link types in `scrapeLink`: map links (`utils/maps.ts`) yield coords + place name; GitHub repo-root links (`utils/github.ts`) skip the HTML scrape and use the GitHub API for `owner/repo` title, description, stars, language, and the README as chunk content (unauthenticated, 60 req/hr; falls back to normal scrape).
3. Search: `tsv` tsvector columns on `Pin`/`PinChunk` (Prisma `Unsupported`, maintained in the DB), queried via `searchPins` raw SQL with prefix matching + snippets.

## Auth

- Web sessions: better-auth (`apps/web/lib/clients/auth.ts`) with `phoneNumber` plugin; OTP via Twilio Verify (`features/twilio/twilio.service.ts`). Route protection is `apps/web/proxy.ts` (Next 16's middleware), which only checks cookie presence and skips RSC requests.
- Public API: `GET /api/v1/pins` with `Authorization: Bearer pm_...`. One key per user, sha256-hashed lookup, all in `apps/web/features/key/key.service.ts`.

## Commands (run from repo root)

- `pnpm dev` / `pnpm build` / `pnpm lint`
- `pnpm db:generate` (Prisma client; turbo runs it before dev/build/lint automatically), `pnpm db:push`, `pnpm db:studio`
- No test suite exists.

## Local dev for agents (seeing the UI)

- `pnpm dev` starts web on :3000 and api on :8080. Drive the web app with a headless browser to verify UI work.
- Login: with `DEV_OTP` set in `apps/web/.env` (it is, locally), enter any phone (e.g. +15555550100) on `/login` and use that code as the OTP. Twilio is skipped entirely; new phones auto-create users. Never set `DEV_OTP` in prod.
- Pin processing: `MessageQueueClient.publish` skips QStash and POSTs directly to the api when `BACKEND_API_URL` is localhost (the local default), so created pins go PROCESSING -> READY end to end locally. Point `BACKEND_API_URL` back at the Railway URL to test the real QStash path.
- The local `.env` DATABASE_URL is the shared Neon db; clean up test pins/users you create.

## Gotchas

- Next.js 16 has breaking changes vs training data; read `node_modules/next/dist/docs/` before writing Next code (see `apps/web/AGENTS.md`). `proxy.ts` replaces `middleware.ts`; web dev runs `next dev --webpack`.
- New env vars must be added to the `env` arrays in `turbo.json` or builds won't see them. Current vars: `DATABASE_URL`, `BETTER_AUTH_SECRET/URL`, `TRUSTED_ORIGINS`, `TWILIO_SID/AUTH_TOKEN/VERIFY_SERVICE_SID`, `QSTASH_*`, `BACKEND_API_URL`, `CORS_ORIGIN`, `ANTHROPIC_API_KEY`, `DEV_OTP` (dev only).
- No migrations; schema changes go out via `db:push`. The `tsv` columns and their triggers live outside Prisma, so don't expect Prisma to manage them.
- Web app has no `src/` dir: code sits in `app/`, `components/`, `features/`, `lib/` directly. Client data fetching is SWR via `lib/requests/PinRequests.ts`; client state is zustand (`lib/stores/`); UI primitives are shadcn in `components/ui/`.
- Pin titles are HTML-entity-decoded with `decodeEntities` on write; keep doing that for any new write path.

---
> Source: [Abhishek-More/pinmy](https://github.com/Abhishek-More/pinmy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
