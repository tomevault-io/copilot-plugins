## yet-another-mtg-database

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A personal Magic: The Gathering card database and collection manager. Next.js 16 (App Router, React 19) frontend + API routes, backed by MongoDB via Mongoose. Card data originates from Scryfall bulk JSON. Features include Scryfall-style search, collection and deck management, drag-and-drop card organization, and camera-based card scanning (proxied to an external card-scanner backend).

## Planning & implementation checklist

Any non-trivial plan must include steps for:
1. **Tests** — add or update unit, integration, component, and/or e2e tests as appropriate for the change. Do not skip this even if the task description doesn't mention it.
2. **Lint** — run `npm run lint` at the end and fix all errors and warnings before considering the task done.

## Commands

```bash
npm run dev              # Start dev server (http://localhost:3000)
npm run build            # Production build
npm run start            # Run production build
npm run lint             # ESLint (eslint-config-next)

npm run init-db          # Import Scryfall bulk card data into MongoDB
npm run whitelist-user   # Whitelist a user by email (see Auth below)

npm test                 # Run all Vitest projects (unit + integration + components)
npm run test:unit        # Pure-logic unit tests (search engine, sort, helpers)
npm run test:integration # API route + server-helper tests (mongodb-memory-server)
npm run test:components  # React component/hook/context tests (jsdom + RTL)
npm run test:coverage    # Vitest with v8 coverage
npm run test:e2e         # Playwright E2E (run `npm run test:e2e:install` once first)
```

### Testing

The suite (added with Vitest 4 + Playwright) lives in three Vitest projects plus E2E:
- **`unit`** (node) — pure functions: `src/lib/search/**`, `src/lib/sortConfig.ts`, `grouping.ts`, etc. Tests co-located in `__tests__/`.
- **`integration`** (node) — API route handlers and `src/lib/server/**` against an in-memory MongoDB (`mongodb-memory-server`); `getServerSession` is mocked. Files in `tests/integration/`; shared lifecycle/auth/seed helpers in `tests/integration/setup.ts` and `helpers.ts`. Runs single-fork with isolation off so one DB connection is shared.
- **`jsdom`** (jsdom) — components, hooks, contexts via Testing Library + MSW. Global mocks (`next/navigation`, `next/image`, `matchMedia`) in `vitest.setup.jsdom.ts`.
- **E2E** (`e2e/`, Playwright) — runs the app on port 3100 with its own `distDir` (`E2E_DIST_DIR`, so it never collides with a running dev server). `e2e/global-setup.ts` starts a fixed-port in-memory Mongo, seeds a user + active "Main Collection" + cards, and mints a NextAuth session cookie (no production auth changes).

Config: `vitest.config.ts`, `playwright.config.ts`. Run a single project with `vitest run --project <name>`.

### Local development infrastructure

`docker-compose-dev.yml` spins up only the backing services — MongoDB (host port `27017`), the card-scanner Postgres DB, and the external `card-scanner` backend (host port `8000`) — **without** the Next.js app. Use it for developing the app locally with `npm run dev` (which connects to Mongo via `MONGO_DB_URI` in `.env`, default `mongodb://127.0.0.1:27017/...`):

```bash
docker compose -f docker-compose-dev.yml up -d   # start databases + scanner
npm run dev                                       # run the Next.js app on the host
```

`docker-compose.yml` is the full production stack (app + Mongo + scanner behind Caddy); `docker-compose-dev.yml` is the same minus the app service and reverse proxy.

### Seeding the database

`init-db` (`src/scripts/init-db.ts`) streams a large Scryfall "oracle/all cards" JSON file into the `cards` collection. It is a `commander` CLI:

```bash
npm run init-db -- -f bulk-data/oracle-cards-XXXX.json   # import from file
npm run init-db -- --data-url <url>                       # download + import
npm run init-db -- -f <file> --clear                      # wipe cards first
```

Place bulk JSON files in `bulk-data/` (gitignored). Scripts run via `tsx` and load env through `dotenv/config`, reading `MONGO_DB_URI`.

### Whitelisting a user

Sign-in is **deny-by-default**: only emails present in the `users` collection can log in (see Auth). Add one with:

```bash
npm run whitelist-user -- user@example.com
```

## Environment

Copy `.env.example` to `.env`. Key vars: `MONGO_DB_URI`, `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` / `AUTH_SECRET` / `NEXTAUTH_URL` (NextAuth Google OAuth), `SCANNER_BASE_URL` (external card-scanner backend the `/api/scan` route proxies to — defaults to `http://localhost:8000`), `ALL_CARDS_FILE` (default bulk import path), `SCRYFALL_API_BASE_URL` (Scryfall API base, defaults to `https://api.scryfall.com`; used by the card-refresh and set-icon routes).

Scryfall requires a custom `User-Agent` and an `Accept` header on every API request, and a max of 10 requests/second — all Scryfall `fetch`es go through `scryfallFetch` / `SCRYFALL_HEADERS` in `src/lib/scryfall.ts`, which attaches the headers and rate-limits starts to <= 10/s via an in-process serialized queue (relies on the server being a long-lived singleton; not coordinated across instances).

## Architecture

### Data layer (`src/db/`, `src/types/`)
- **`src/db/mongoose.ts`** — `connectDB()` caches the connection on `global.mongoose` to survive Next.js hot-reload. **Every API route must `await connectDB()` before touching a model.**
- **`src/db/schema.ts`** — all Mongoose schemas/models, guarded with the `mongoose.models.X || mongoose.model(...)` pattern (required so re-imports during hot-reload don't throw). Models: `CardData`, `PhysicalCardModel`, `CollectionModel`, `DeckModel`, `UserModel`, `TagModel`, `SetSvgModel`.
- Plain TS interfaces in `src/types/` are the source of truth for document shapes; schemas are typed against them. The `MtgCard` shape mirrors Scryfall's card JSON.

#### Physical-card-instance model (collections & decks)
This is the core domain model. Read it before touching collections/decks/drag-and-drop.
- **`CardData`** (`src/types/MtgCard.ts`) — Scryfall reference card data. The model is named `CardData` to disambiguate from physical copies, but is **pinned to the Mongo collection `cards`** (`{ collection: "cards" }` in the schema) so the bulk import and `$lookup`s stay stable. Cards are referenced everywhere by Scryfall string `id`, **not** Mongo `_id`.
- **`PhysicalCard`** (`src/types/PhysicalCard.ts`) — one document per physical card copy: `{ owner, cardId, collectionId (required), deckId? (optional), notes?, tags? }`. **Invariant: every physical card belongs to exactly one collection and is assigned to at most one deck.** The `collectionId`/`deckId` back-refs are the **source of truth for membership** and power the cross-membership badges.
- **`Collection`** (`src/types/Collection.ts`) — `{ owner, name, description, isActive? }`. No stored card order: membership is purely `PhysicalCard.collectionId`, and the collection table groups + sorts deterministically client-side.
- **`Deck`** (`src/types/Deck.ts`) — `{ owner, name, description, sections: [{ name, columns: [{ cards: ObjectId[] }] }] }`. A deck is an *arrangement* layered over physical cards that still live in their collections: named sections, each with any number of unnamed columns, each an ordered list of `PhysicalCard` ids. Sections/columns get auto `_id`s used by the UI/API.
- **No multi-doc transactions** (dev Mongo is a single mongod). Mutations write the `PhysicalCard` back-ref **first**, then fix up the deck's ordered arrays. `GET /api/decks/[id]?details=true` **reconciles** by appending any card whose `deckId` points at the deck but is missing from the arrays into a default column — so a mid-failure is always recoverable. Shared helpers: `src/lib/server/cardDetails.ts` (`detailPhysicalCards`, `upsertTags`) and `src/lib/server/deckArrange.ts` (`findOrCreateColumn`, `pullCardFromAllDecks`).
- API surface: `/api/collections` (CRUD + `summaries` + `isActive`), `/api/decks` (CRUD + `/sections`, `/columns`, `/cards` placement ops `place|move|remove`), `/api/physical-cards` (POST create-N / PATCH notes·tags·collection / DELETE / `remove-group` decrement). The old `/api/collections/[id]/cards` action API is gone.
- "Quantity" is a **display-only** concept now: the collection table groups copies by `cardId + notes + tags + deckId` (see `src/components/my-cards-page/collection-view/grouping.ts`) into one row with a count and a single deck badge (loose copies sort before deck-assigned ones). There is no `quantity` field on any document.

### Search engine (`src/lib/search/`)
Parses Scryfall-like query strings into MongoDB query objects. Entry point: `parseSearchQuery(queryString)` (re-exported from `index.ts`). Pipeline:
1. **`parser.ts`** — `tokenizeQuery` (handles quotes, parentheses) then `parseTerm` (splits `key:value`, comparison operators `>= <= > < =`, and `-` negation).
2. **`queryBuilder.ts`** — recursive-descent `parseExpression` builds boolean structure: implicit AND between terms, explicit `or`, parenthesized groups, and `$nor` for negation. Bare terms (no `key:`) become a name/`flavor_name` regex search.
3. **`config.ts` + `operators/`** — each operator (color, type, oracle, manavalue, set, rarity, etc.) is a `SearchOperatorConfig` with `aliases`, `buildQuery(value, operator)`, and optional `validate`. **To add a search operator: create a file in `operators/`, export it from `operators/index.ts`, and register it in `searchOperators` in `config.ts`.**

Sorting is separate (`src/lib/sortConfig.ts`): some sort fields require a MongoDB aggregation pipeline (`useAggregation` + `buildAggregationSort`). `GET /api/cards` switches between `.find().sort()` and an aggregation pipeline, also using `$lookup` against `physicalcards` (localField `id` → foreignField `cardId`) for the `owned=true` filter.

### Auth (`src/auth.ts`)
NextAuth (v4) with Google provider, **JWT session strategy (no DB session store)**. `src/app/api/auth/[...nextauth]/route.ts` just re-exports the handler.
- The `signIn` callback enforces the whitelist: rejects any email not found in `users`. On first successful sign-in it auto-creates an **active** "Main Collection" (`CollectionModel`, `isActive: true`) so that search→deck drops work out of the box (adding a card to a deck requires an active collection to own the new physical card).
- The DB `_id` is threaded through `jwt` → `session` callbacks and exposed as `session.user._id` (typed via module augmentation in `auth.ts`).
- **API routes are auth-gated by `src/proxy.ts`** — this is the Next.js 16 middleware (the file was renamed from `middleware.ts` to `proxy.ts` in Next 16). Its `getToken()` check returns `401 { error: "Unauthorized" }`, and its `matcher` (`"/api/((?!auth).*)"`) protects every `/api/*` route except `/api/auth/*`. Routes that need the user id additionally call `getServerSession(authOptions)` and read `session!.user._id` (trusting the middleware).
- Pages are also gated server-side: e.g. `(with-app-bar)/(main)/layout.tsx` calls `getServerSession` and redirects to `/login` when absent.

### Routing (`src/app/`)
App Router with route groups (folders in parentheses don't affect URL paths):
- `(with-app-bar)/` — adds the global `AppBar`. `(main)/` nested group adds auth gate + `MainWorkspace`, containing `/search` and `/my-cards`.
- `/my-cards` is a landing page (create/list collections + decks). Collections and decks are **distinct** entities with separate detail pages: `/my-cards/collections/[id]` (→ `CollectionTable`) and `/my-cards/decks/[id]` (→ `DeckView`). There is no longer a single page that toggles between table and deck views.
- `/scan` + `/scan/results` — camera capture and recognition results (client-side, uses `ScanContext`).
- API routes under `app/api/`: `cards`, `collections`, `decks`, `physical-cards`, `sets`, `tags`, `scan`, `auth`.

### Card scanning (`src/app/api/scan/`, `src/app/scan/`)
`POST /api/scan` (`src/app/api/scan/route.ts`) is a thin **auth-guarded proxy** to the external card-scanner backend (`ghcr.io/andrew-meads/card-scanner-backend`, defined in the compose files at port 8000). It validates the multipart `image` field, forwards it to `${SCANNER_BASE_URL}/api/scan` (default `http://localhost:8000`), and passes the scanner's JSON back verbatim — each detected card's de-skewed crop plus a pre-ranked list of candidate Scryfall printings (`{ count, cards: [{ id, url, width, height, matches }], debugUrl }`, typed in `src/types/ScanResult.ts`). Returns `502` if the scanner is unreachable. Touches no Mongo model.

The client flow lives in `src/app/scan/`:
- `/scan` (`page.tsx`) — camera capture (full-frame, multi-card; front/back `facingMode` toggle; best-effort tap-to-focus) **or** uploading an existing image; both POST the blob via `usePostImageForScan` → `/api/scan`, store the result in `ScanContext`, and route to `/scan/results`.
- `/scan/results` (`page.tsx` + `src/components/scan/ScannedCardItem.tsx`) — per detected card, shows the crop and a horizontally-scrolling, single-select strip of candidate printings (best match pre-selected), a quantity stepper, and an Add button that adds the chosen printing **by its `scryfallId`** to the active collection (`useCreatePhysicalCard` with `quantity`, creating N physical cards) without leaving the page. Closing does `history.go(-2)` (past `/scan`).
- Crops come back as scanner-relative `url`s (`/cards/<file>.jpg`) and are served through the auth-proxied **`GET /api/scan/crops/[file]`** route. Render them with a plain `<img>` — `next/image` can't optimize this route because the optimizer fetches server-side without the user's auth cookie (→ 401). Candidate images are `cards.scryfall.io` URLs (allowed in `next.config.ts`), so `next/image` is fine there.

### Set icons (`src/app/api/sets/[code]/svg/route.ts`)
Lazily caches Scryfall set-symbol SVGs into the `setsvgs` collection on first request, then serves from DB with long-lived cache headers.

### Client state & data fetching
- **TanStack Query** for all server state. Hooks live in `src/hooks/react-query/` (e.g. `useInfiniteCardsSearch`, `useRetrieveCollectionDetails`, `useRetrieveDeckDetails`, `useCreatePhysicalCard`, `useDeckCardOp`). Query keys: `["collection-summaries"]`, `["collection-details", id]`, `["deck-summaries"]`, `["deck-details", id]`, `["card-locations", name]`, `["tags"]`. Cross-kind moves (deck placement ↔ collection membership) change a badge on the "other" side, so the physical-card/deck mutations broadly invalidate both `["collection-details"]` and `["deck-details"]` plus `["card-locations"]` (see `src/hooks/react-query/invalidate.ts`). Provider in `src/context/QueryProvider.tsx`; all providers composed in `src/context/Providers.tsx`.
- **React Context** for cross-component UI state: `CardSelectionContext`, `OpenEntitiesContext` (holds the user's open collections **and** decks as a `kind`-discriminated `OpenEntitySummary[]`; only collections can be `active`), `ScanContext`.
- **react-dnd** (HTML5 backend) for drag-and-drop; sources/targets in `src/hooks/drag-drop/`. Two drag item types: `NEW_CARD` (from search) and `PHYSICAL_CARD` (one or more existing copies). All drop targets delegate to a single dispatcher, **`useDropDispatch`**, which implements the six drag scenarios: search→collection (create), search→deck (create in active collection, then place — errors via toast if no active collection), collection↔collection (change `collectionId`), collection/deck→deck (place, clearing any prior deck), deck→collection (clear `deckId`, optionally change collection). The collection table shows each card's deck badge; the deck view shows each card's collection badge. The collection table is **virtualized** with `@tanstack/react-virtual` (no pagination, no manual row reorder).
- UI is **shadcn/ui** (Radix primitives) in `src/components/ui/` + **Tailwind CSS v4** (config in `globals.css` / `postcss.config.mjs`, no `tailwind.config`). `mana-font` renders mana symbols.

## Conventions
- Import alias `@/*` → `src/*`.
- Prettier: no tabs, double quotes, no trailing commas, 100-col width.
- Card identity across collections is the Scryfall string `id`, never the Mongo `_id`.
- Remote images are restricted to specific hosts in `next.config.ts` (`cards.scryfall.io`, `errors.scryfall.com`, `lh3.googleusercontent.com`) — add new hosts there.

## Deployment
Multi-stage `Dockerfile` (Node 22) + `docker-compose.yml` running the app, MongoDB, and the external card-scanner stack behind a Caddy reverse proxy (labels in compose). For local development use `docker-compose-dev.yml` instead (backing services only — see Commands).

---
> Source: [andrew-meads/yet-another-mtg-database](https://github.com/andrew-meads/yet-another-mtg-database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
