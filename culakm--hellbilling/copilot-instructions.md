## hellbilling

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HellBillingRB is a web app for creating and managing car journey roadbooks. Users create **trips** containing ordered **lines** (waypoints), with map visualization, PDF export, and role-based access (user/editor/admin). Full spec in `DOC/Spec.md`.

## Tech Stack

- **Vue 3** (Composition API, `<script setup>`, no TypeScript — pure JS with ES modules)
- **Quasar 2** UI framework with Material Icons, Sass theming (`src/css/quasar-variables.sass`)
- **Pinia 3** setup-style stores (auth, trips, lines, users)
- **Vue Router 4** with auth navigation guards and lazy-loaded routes
- **Firebase** backend: Auth, Firestore, Storage, Cloud Functions v2 (Node 24), App Check
- **Vite 7** build tool, dev server on port 5175
- **vue3-google-map** for Google Maps with Advanced Markers
- **pdfMake + html-to-pdfmake** for PDF export (used in `usePdfExport` composable)
- **vue-draggable-plus** for line drag-and-drop reordering

## Commands

```bash
npm run dev              # Vite dev server (port 5175)
npm run build            # Production build → dist/
npm run fb-emul          # Firebase emulators with seed data
npm run fb-emul-exit     # Emulators with data export on exit
npm run fb-emul-empty    # Emulators without seed data
npm run fb-web-up        # Deploy to Firebase Hosting + Functions
npm run fb-web-down      # Disable Firebase Hosting
```

## Firebase Emulator Ports

| Service   | Port |
| --------- | ---- |
| Auth      | 9100 |
| Firestore | 8081 |
| Functions | 5002 |
| Storage   | 9200 |
| UI        | 4001 |

Emulator auto-detection in `src/firebase.js` checks `http://localhost:4001/emulator/v1/projects` at startup.

## Architecture

### Firestore Data Model

```
users/{userId}
trips/{tripId}
trips/{tripId}/lines/{lineId}
```

Roles (user/editor/admin) are stored as **custom claims** on Firebase Auth tokens, not in Firestore documents. No self-registration — admins create accounts.

### Frontend Structure

- `src/pages/` — route pages organized by domain (auth, trips, users, test)
- `src/components/` — reusable components by domain (trips, lines, users, maps, ui)
- `src/stores/` — Pinia setup-style stores (auth, trips, lines, users)
- `src/composables/` — shared logic (useError, useFirebaseStorage, useFormValidationRules, usePdfExport, useUtils)
- `src/utils/coordinates.js` — DMS/decimal coordinate conversion
- `src/config/settings.js` — app config (interest categories, closeLineThresholdKm)
- `src/layouts/` — MainLayout with TheHeader, TheDrawer, TheFooter

### Cloud Functions (`functions/`)

- `functions/scripts/users/` — onCall: createUser, updateUser, deleteUser
- `functions/scripts/auth/` — role checking utilities, getUserRole
- `functions/scripts/trips/` — Firestore triggers: increment/decrement linesCount on line add/delete
- `functions/scripts/services/` — helloWorld health check, firstUserRole bootstrap

### Path Alias

`@` resolves to `./src` (configured in `vite.config.js` and `jsconfig.json`).

### Key Patterns

- All components use `<script setup>` with Composition API
- Stores use `storeToRefs()` for reactive destructuring
- Routes use `meta.requiresAuth` / `meta.requiresUnauth` enforced by navigation guards
- Environment variables via `.env.local` accessed as `import.meta.env.VITE_*`
- Google Maps uses separate API keys for production and localhost
- Auth state is restored from Firebase via `onAuthStateChanged` in `src/stores/auth.js`; the router guard `await`s `authStore.authReady` before evaluating `meta.requiresAuth`, so page `onMounted` data loads can assume `auth.currentUser` is populated. `main.js` calls `useAuthStore()` between `app.use(pinia)` and `app.use(router)` to wire the listener before the first guard runs. 24-hour idle auto-logout still applies.
- Rich-text HTML fields (e.g. `line.note`, authored via `mcQEditor`) must be rendered through `sanitizeRichText` from `@/composables/useSanitize` before `v-html` — DOMPurify-based allowlist sanitization is the only thing standing between stored HTML and the DOM

---
> Source: [culakm/hellbilling](https://github.com/culakm/hellbilling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
