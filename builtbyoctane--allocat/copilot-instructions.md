## allocat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

AlloCat — minimalist personal-finance PWA, also shipped as a native Android app (Capacitor) for SMS-based transaction tracking. Next.js 16 (App Router) + React 19 + Supabase + Dexie (IndexedDB) + TanStack Query 5 + Tailwind v4. Currency is multi-currency via `lib/number-format.ts` + `lib/currency/catalog` (INR is the default for legacy callers); do **not** assume hardcoded `en-IN`.

`package.json` declares `name: "AlloCat-web"` despite the directory name.

## Commands

```bash
npm run dev         # next dev
npm run build       # next build && serwist build (service worker bundling)
npm run start       # next start
npm run lint        # eslint (flat config in eslint.config.mjs)
npm run test        # vitest run (one-shot)
npm run test:watch  # vitest (watch)

npx vitest run lib/sms/match.test.ts        # single file
npx vitest run -t "matches exact rule"      # single test by name
```

Test files live next to source (`*.test.ts`, e.g. `lib/ai/parseSmsTransaction.test.ts`, `lib/sms/match.test.ts`). No typecheck script — run `npx tsc --noEmit` if needed.

Use **pnpm** (per memory: `npm install` fails with ERESOLVE). Both `package-lock.json` and `pnpm-lock.yaml` are checked in.

### Android (Capacitor)

```bash
npx cap sync android                                  # copy web config + plugins into android/
CAP_SERVER_URL=http://192.168.1.20:3000 npx cap sync # point the shell at a LAN dev server
```

Requires Android Studio JBR (JDK 21). Open `android/` in Android Studio to build/run the APK.

## Required env (`.env.local`)

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY=
OPENROUTER_API_KEY=     # used by app/api/ai/chat
```

## Architecture

### Offline-first sync (the central pattern — read this before touching data flow)

Every page reads from IndexedDB first; the network is a fallback and a background reconciler. Three layers cooperate:

1. **IDB cache** — `lib/db/AllocatDB.ts` defines a Dexie schema mirroring the Supabase tables (`profiles`, `budgets`, `categories`, `budget_items`, `goals`, `assets`, `asset_categories`, `asset_value_history`, `debts`, `reports`, `net_worth_snapshots`, `activity_logs`, `merchant_rules`, `sms_transactions`) plus three sync infra tables: `sync_queue`, `id_map`, `sync_meta`. Currently at schema version 10. The DB is a browser-only singleton via `getDB()` — calling it on the server throws. Schema version bumps live in `AllocatDB.ts` constructor; add a new `.version(N).stores({...})` block, never mutate prior versions.

2. **Hydration + prefetch** — on mount, `SyncProvider` (`lib/providers/SyncProvider.tsx`) calls `hydrateAllTables()` (`lib/db/hydrate.ts`) which bulk-pulls every table for the current user into IDB. If `sync_meta.__userId__` differs from the active user, IDB is wiped first (multi-account device safety). After hydration, `prefetchAllQueries()` (`lib/db/prefetch.ts`) warms the React Query cache from IDB so first navigation has no skeletons. Use `qc.fetchQuery` (not `prefetchQuery`) when adding new prefetched keys — staleTime semantics would otherwise serve stale entries across reloads.

3. **Mutation queue** — mutations write to IDB optimistically (with a `temp_<uuid>` id for INSERTs), then `useEnqueue()` appends a `SyncQueueItem` to `sync_queue`. `SyncEngine` (`lib/sync/SyncEngine.ts`) drains the queue: each `(table, operation)` pair maps to a server action via the `dispatch` table — when adding new tables/operations, you must register a dispatcher entry there or the item will permanently fail. Failed items retry up to `MAX_RETRIES = 3` with backoff; permanent failures invoke `onRollback` (which invalidates relevant React Query keys). `temp_` ids inside payloads are rewritten to real ids via `id_map` before the action fires — use `extractTempIds` patterns when designing new payloads.

Cross-cutting rules:
- Server actions live in `lib/actions/<domain>.ts` and are the *only* path that talks to Supabase from the client side. They are also called directly during initial fetch (IDB miss) and via SyncEngine on flush.
- Read hooks live in `lib/hooks/use<Domain>.ts`. The pattern is: `getXFromIDB()` first; on miss, fall back to the server action. Each hook exports its query key constant (e.g. `DASHBOARD_KEY`, `budgetKey(month, year)`) — reuse these for invalidation.
- Mutation hooks must: (1) write to IDB optimistically, (2) `enqueue` the operation, (3) invalidate matching query keys in `onSuccess`.

### Routing

- `app/(app)/*` — protected app shell (dashboard, budget, debt, goals, net-worth, profile, activity, **sms**). Layout wraps in `TourProvider` → `SyncProvider`, with mobile-first 480px frame and `md:` desktop layout.
- `app/auth/*` — login / signup / oauth callback.
- `app/onboarding/page.tsx` — post-signup flow.
- `app/share-target/` — PWA Web Share Target landing (manifest `share_target` posts here); shared text is parsed by `lib/ai/parseSpend.ts`.
- `app/api/ai/chat/route.ts` — streaming AI chat. Hard off-topic regex guard runs *before* the model call; topic detection in `lib/ai-utils.ts` decides which slice of `buildFinancialContext` to attach.

### Auth + middleware quirk

Auth uses `@supabase/ssr` with cookie-based sessions:
- `lib/supabase/server.ts` — server actions / RSCs
- `lib/supabase/client.ts` — browser
- `lib/supabase/middleware.ts` — `updateSession` refreshes tokens and gates routes

**Note**: The Next.js middleware file is named `proxy.ts` (not `middleware.ts`), exports a `proxy` function, and lives at the repo root. Do not rename it without verifying the Next 16 convention — both forms have existed across versions.

Protected paths (redirect to `/auth/login` if no user): `/dashboard`, `/budget`, `/net-worth`, `/debt`, `/onboarding`. `/goals`, `/profile`, `/activity` are *not* in this list — confirm intent before adding new private routes.

### Activity log

Server actions write to `activity_logs` via `lib/server/activity-logger.ts` (`logActivity` + `fmt` for INR formatting). Per memory: the SQL migration for the `activity_logs` table still needs to be run on Supabase if missing.

### Onboarding tour

Driver.js tour managed by `lib/tour/` — `TourContext` persists `{ enabled, seenPages }` in `localStorage` under `allocat-tour-state`. Add new pages by extending `tourSteps.ts` + `types.ts`.

### Native Android + SMS transaction tracking (Android-only)

The native app runs in **remote-URL WebView mode** (`capacitor.config.ts`): the shell loads the deployed Next.js app over the network, so SSR, server actions and the offline-first IDB layer work unchanged — no web assets are bundled (`webDir: "public"` is a CLI formality). `components/pwa/NativeShell.tsx` calls `SplashScreen.hide()` once mounted (auto-hide is disabled so users don't see a blank WebView during load).

The core native feature reads incoming bank/UPI **transaction** SMS and auto-categorizes spends. Full design in `docs/sms-feature.md`. Pipeline:

1. **Native receiver** (`android/app/src/main/java/app/allocat/mobile/`) — `SmsTransactionReceiver` fires even when the app is killed. `SmsFilter` applies an on-device financial-only gate (Play SMS-policy compliance). Messages are queued in `SmsQueue` (SharedPreferences); if the WebView is foregrounded, a `smsReceived` event is emitted. When app is closed, `SmsParser` (a Java port of the TS parser) + `SmsNotifier` post a transaction notification directly. **Only `RECEIVE_SMS` is declared — the app never reads the existing inbox (`READ_SMS` is intentionally absent).**
2. **JS bridge** — `components/pwa/SmsBridge.tsx` (native-only) listens for live events and drains the queue silently on open (native already notified). It mirrors merchant rules / quick-allocate targets / notif config into native via the `SmsReader` Capacitor plugin (`lib/native/SmsReader.ts`).
3. **Ingest** — `lib/sms/ingestClient.ts` parses on-device (`lib/ai/parseSmsTransaction.ts`, regex, **no LLM/network** — removed for Play compliance), matches a learned `merchant_rules` row (`lib/sms/match.ts`, exact > contains > regex), writes an optimistic `sms_transactions` IDB row, and enqueues a sync INSERT. **Privacy: only extracted fields + a hashed dedupe key sync to the server; the raw SMS body/sender stay on-device.** Only debits are tracked; credits are ignored.

Keep `SmsParser.java` regex in sync with `lib/ai/parseSmsTransaction.ts` (both are documented as needing to match). Notifications go through `lib/native/notify.ts` (`notifyLocal`, no-op on web); sounds in `android/app/src/main/res/raw/` mapped by `lib/native/notifSounds.ts`.

### PWA

**Serwist** (`@serwist/next`), not next-pwa. The service worker source is `app/sw.ts`, configured via `serwist.config.js`; `npm run build` runs `serwist build` after `next build`. Disabled in dev. Manifest at `app/manifest.ts` (includes `share_target` and shortcuts). Install prompt UI in `components/ui/InstallPrompt.tsx`.

## Path alias

`@/*` → repo root (see `tsconfig.json`). Use it for all internal imports.

## Design system

Neo · Lime redesign with a runtime **accent system**: presets in `lib/theme/accents.ts` (`lime` default, plus tangerine/lemon/purple/blue). `data-accent` on `<html>` swaps tokens defined in `app/globals.css`; `AccentProvider` (`lib/providers/AccentProvider.tsx`) mirrors/persists the choice (`allocat-accent`), but the no-flash initial paint is a blocking script in `app/layout.tsx`. Light/dark via `next-themes` (`ThemeProvider`). Chart colors in `lib/theme/dataViz.ts`.

Fonts (root layout): Hanken Grotesk (`--font-sans`), Bricolage Grotesque (`--font-display`), JetBrains Mono (`--font-mono`). Material Symbols Outlined from Google Fonts. Tailwind v4 (PostCSS plugin in `postcss.config.mjs`, no `tailwind.config.*`).

---
> Source: [BuiltByOctane/allocat](https://github.com/BuiltByOctane/allocat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
