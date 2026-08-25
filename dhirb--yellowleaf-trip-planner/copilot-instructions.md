## yellowleaf-trip-planner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Yellowleaf is a calm, large-type travel itinerary app, backed by Firebase. It has **two personas served by one SPA**:

- **Traveler** (`/t/:tripId`) — opens a shared/published trip (private trips gated client-side by an access code), browses Today / Calendar / Details. Read-mostly, offline-capable PWA.
- **Admin** (`/admin`) — email/password sign-in, a trips list, and a live trip editor (Itinerary + Settings tabs) with autosave and an AI assist.

Both routes are lazy-loaded in `src/App.tsx`. Everything renders inside `AppShell` — a fixed-viewport, mobile-first single column capped at `max-w-[440px]` (a "phone frame"); screens own their internal scrolling.

## Commands

```bash
npm run dev            # Vite dev server (http://localhost:5173 → /admin)
npm run build          # tsc -b + vite build → dist/
npm run lint           # oxlint
npm run format         # prettier --write .
npm test               # boots Firestore emulator, runs Vitest rules/unit tests once
npm run test:watch     # same, watch mode
npm run test:e2e       # Playwright E2E (test/e2e/)
npm run emulators      # Auth + Firestore + Hosting emulators
npm run deploy         # build + firebase deploy (hosting + firestore)
```

Run a single test file / case:

```bash
firebase emulators:exec --only firestore "npx vitest run test/editTrip.test.ts"
firebase emulators:exec --only firestore "npx vitest run -t 'soft-delete'"
```

> `npm test` wraps Vitest in `firebase emulators:exec`, so the emulator is always up for the run. The emulator launcher **requires JDK 21+ on PATH** — if your default `java` is older: `JAVA_HOME=$(/usr/libexec/java_home -v 21+) npm test`.

Provision an admin account (custom-claim `role: admin`, needs `GOOGLE_APPLICATION_CREDENTIALS`):

```bash
npm run provision -- user@example.com [admin]
```

## Tech stack

Vite 8 · React 19 · TypeScript 6 · Tailwind v4 (via `@tailwindcss/vite`, no config file — tokens live in `src/styles/tokens.css`) · react-router-dom 7 · Firebase (Auth, Firestore, Hosting, App Check, AI Logic/Gemini) · `vite-plugin-pwa` (Workbox). Linting is **oxlint**, not ESLint.

## Architecture (the parts that span files)

**Data layer — `src/lib/trips.ts`.** All Firestore access goes through here; components never touch `firebase/firestore` directly. One document per trip in the `trips` collection; `days`, `flights`, `items`, `contacts`, etc. are **embedded arrays** (trips are small). Every read passes through `toTrip()` → `normalizeTrip()` and timestamps are normalised to epoch millis.

**Soft-delete contract.** `allow delete: if false` for everyone in `firestore.rules`. "Deleting" a trip sets `deletedAt` via an update (`softDeleteTrip`); the data layer then treats it as missing in every read path (`getTrip`, `subscribeTrip`, list). Never write a hard delete — it will be rejected and breaks the contract that `test/firestore.rules.test.ts` asserts end-to-end.

**Immutable edits — `src/lib/editTrip.ts`.** Pure functions that take a `Trip` and return a **new** `Trip` (each `structuredClone`s first). This is the only sanctioned way to mutate trip content. Follow the existing function shape when adding edit operations; never mutate a trip in place in a component.

**Editor state — `src/hooks/useTripEditor.ts`.** Subscribes to the trip live **and** autosaves on a 600ms debounce. Critical invariant: remote snapshots are applied only when the working copy is **clean** (`dirty.current` / `savingRef.current` both false), so live echoes never clobber unsaved admin edits. Preserve this dirty-tracking if you touch it.

**Ownership.** `ownerId` (full control) plus `coOwnerEmails` (edit content, can't delete or re-share). `subscribeOwnerTrips` runs the owned + co-owned queries in parallel, fault-isolated: the owned query is primary, the co-owned query degrades gracefully so a co-ownership hiccup can never hide trips you own.

**Auth & roles — `AuthProvider` + `useAuth`.** Admin status comes from a Firebase **custom claim** (`role`) read off the ID token, not a Firestore lookup. `loading` stays true until the role resolves, so consumers never see a signed-in user with undetermined admin status.

**i18n — `src/lib/localize.ts`.** English is the implicit base. Every translatable entity (`Item`, `Stay`, `Day`) carries an optional `t` map of per-language overrides; `localize*` merges defined, non-empty overrides over the base at read time. A blank/missing override never clobbers the English source.

**AI assist — `src/lib/ai.ts`.** Firebase AI Logic (Gemini text, grounded with Google Search), **dynamically imported** so its weight stays out of the initial bundle. Thumbnails come from a Wikimedia search (`src/lib/imageSearch.ts`), which returns null rather than throwing, so a miss leaves the thumbnail blank instead of inventing an image. `describeAiError` / `aiFailureReason` are the single place raw SDK failures become admin-facing copy (App Check and quota get named explicitly) — route new AI error paths through them. Not emulated — it calls the live API even under the emulators.

**Firebase init — `src/firebase.ts`.** App Check (reCAPTCHA v3) attests requests; skipped under Node/Vitest, and gated on a site key so `npm run dev` still runs without one — but **`npm run build` fails without the key** (guard in `vite.config.ts`), because Firebase enforces App Check on every AI Logic request from 2026-11-02 and that cannot be undone. Dev/e2e use a registered debug token (`VITE_FIREBASE_APPCHECK_DEBUG_TOKEN` / `FIREBASE_APPCHECK_DEBUG_TOKEN`); that branch is stripped from production builds. Enforcement is deliberately **AI Logic only** — Firestore and Auth stay unenforced. Firestore uses `persistentLocalCache` (IndexedDB) for offline reads, falling back to in-memory where IndexedDB is absent (tests). `VITE_FIREBASE_*` config is public-by-design (data is protected by rules, not by hiding config).

## Conventions

- **Theme:** light/dark via `ThemeProvider` + `useTheme`; style tokens are CSS custom properties in `src/styles/tokens.css`, surfaced to Tailwind as semantic classes (`bg-app-bg`, `text-faint`, etc.). Add new colours as tokens, not hardcoded values. Dark mode currently ships without first-paint flash handling (intentional, deferred).
- **Layering:** `lib/` = pure logic + data access (no React); `hooks/` = stateful glue; `components/{admin,traveler,ui}/` = views. Keep `lib/` functions pure and React-free.
- **Legacy data:** old Firestore docs and seed literals flow through `src/lib/migrateTrip.ts` (`normalizeTrip`) on every read — extend it (not call sites) when the schema evolves, and add a case to `test/migrateTrip.test.ts`.
- **PWA service worker** is registered programmatically from `src/main.tsx` (`virtual:pwa-register`) to stay CSP-safe — never add inline `<script>`. Hosting CSP + security headers live in `firebase.json`; verify with `curl -I` after deploy.
</content>

</invoke>

---
> Source: [dhirb/yellowleaf-trip-planner](https://github.com/dhirb/yellowleaf-trip-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
