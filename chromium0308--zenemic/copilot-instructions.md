## zenemic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

**Zenemic** is an AI event-planning app: describe an event in natural language and it extracts the
details, builds a planner timeline, and wires up Google Calendar, a Stripe payment split, and
Maps links. The repo holds three parts (the Expo app is now **wired to the backend**; the
keyboard prototype is not yet):

- **`main-app/`** — the shipping **Expo / React Native** iOS app (the product UI).
- **`backend/`** — an **npm-workspaces monorepo** with two runnable services + a shared library.
- **`keyboard/`** — an HTML/JSX **design prototype** of the custom-keyboard UX (reference only).
- `.design-pkg/` — git-ignored design handoff bundle (the source the UI is built from).

### ⚠️ Naming trap — read this first
There are **two `main-app` and two `keyboard` directories** that mean different things:

| Path | What it is |
| --- | --- |
| `main-app/` (repo root) | the Expo **frontend** app |
| `backend/main-app/` | the backend **API service** (`@zenemic/main-app`) |
| `keyboard/` (repo root) | the keyboard **design prototype** (frontend) |
| `backend/keyboard/` | the keyboard **backend service** (`@zenemic/keyboard`) |

Always check whether a path is under `backend/` before assuming what `main-app`/`keyboard` refers to.

## Commands

There is **no automated test suite** anywhere in this repo — `typecheck` is the gate. Don't invent test commands.

### Frontend — run from `main-app/`
```sh
npm install
npm start                 # = expo start; then press i (iOS), a (Android), w (web)
npm run ios | android | web
npm run typecheck         # tsc --noEmit
```

### Backend — run from `backend/` (npm workspaces)
```sh
npm install               # installs all workspaces, symlinks @zenemic/shared
npm run prisma:generate   # REQUIRED before typecheck/build — generates the Prisma client
npm run prisma:migrate    # apply schema (packages/shared/prisma/schema.prisma)
npm run seed              # optional demo user + event
npm run dev:main          # main-app API   → http://localhost:4000
npm run dev:keyboard      # keyboard svc   → http://localhost:4100
npm run typecheck         # all three packages (the gate)
npm run build             # builds shared FIRST, then both services (order matters)
npm run start:main | start:keyboard   # run built dist output
npm run prisma:studio     # inspect the database
```
- Run a script in one package: `npm run <script> -w @zenemic/<shared|main-app|keyboard>`.
- Prisma scripts execute inside `packages/shared` (where `schema.prisma` lives); the root `prisma:*`
  scripts delegate there via `-w @zenemic/shared`.
- Windows gotcha: `prisma generate` can fail with `EPERM` renaming `query_engine-windows.dll.node`
  if a stale `node`/`tsx` server still holds it — kill leftover backend node processes and retry.

## Backend architecture (the part that needs multiple files to grasp)

**Mental model: the two services do NOT call each other over HTTP.** They integrate by sharing
**code** (`@zenemic/shared`) and **state** (one Postgres database). An event created by the keyboard
service appears in the main app because both run the *same* domain code against the *same* DB — not
because one calls the other.

- **`packages/shared` (`@zenemic/shared`) is the brain.** It owns the Prisma client + schema, Supabase
  token verification, the Anthropic AI pipeline (`src/ai`), the integration clients (`src/integrations`:
  Calendar/Maps/Stripe/storage/email/push), and the **domain services** in `src/domain`
  (`events.service` = extract/create/getEvent, `resources.service` = the chart/calendar/split/links
  pipeline, `splitter.service`, `profile`). Everything is re-exported from `src/index.ts`; consumers
  import from `@zenemic/shared`, never by deep path.
- **`main-app` and `keyboard` are thin HTTP layers** — only routes/controllers/entry/env live there.
  `main-app` serves the full `/api/*` surface (auth, events, chat, payments, album, integrations,
  Stripe webhook) on **4000**. `keyboard` serves bare `POST /generate`, `POST /confirm`, `GET /health`
  on **4100**; its controller is the same `extractDraft → createEvent → sendSplitRequests/calendar`
  flow the main app uses, sourced from shared.
- **One database, one client.** `packages/shared/prisma/schema.prisma` + `src/lib/prisma.ts`; both
  services read the same `DATABASE_URL`. Money is stored as integer **minor units** (`lib/money.ts`).
- **Auth.** Both services verify Supabase access tokens (`lib/supabase.ts`) and key a `User` row off
  the Supabase UUID; `ensureProfile` creates it lazily. So the same logged-in user owns events from
  either service.
- **AI structured output** uses *forced tool-use* (the pinned `@anthropic-ai/sdk` 0.69 predates
  `messages.parse`); `src/ai/client.ts` is the single place to change models/SDK. Default model
  `claude-opus-4-8`. Resource generation is best-effort and isolated per resource.
- **Feature flags.** Optional integrations are gated by env keys; missing keys make endpoints return
  `503 not_configured` instead of crashing (`config/env.ts` `features`, surfaced at `/health`).

### Env & config (single repo-root file)
The **whole repo** shares ONE env file at the repository root — `.env.local` (falling back to `.env`).
The backend reads it via `packages/shared/src/config/env.ts` (resolved relative to that file's
`__dirname` → repo root, so it works from any cwd); the Expo app reads it via `main-app/app.config.js`
(loads it with `dotenv` and exposes only the `EXPO_PUBLIC_*` values through `Constants.expoConfig.extra`
— backend secrets are NOT bundled into the app). Copy `.env.example` → `.env.local`. Backend needs
`DATABASE_URL` (+ `DIRECT_URL`), `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `APP_SECRET`,
`ANTHROPIC_API_KEY`; the app needs `EXPO_PUBLIC_API_URL` (your PC's LAN IP for a physical phone),
`EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY`. Ports are **`MAIN_APP_PORT`** (4000) and
**`KEYBOARD_PORT`** (4100) — distinct vars so they never collide; don't reintroduce a bare `PORT`.

### TypeScript build setup
Workspace TS uses `tsconfig.base.json` (`module`/`moduleResolution`: **NodeNext**). Each service has
**two** configs: `tsconfig.json` (dev + `typecheck`, `noEmit`, with a `paths` alias mapping
`@zenemic/shared` → shared **source** so `tsx`/typecheck need no prebuild) and `tsconfig.build.json`
(emit, `paths` reset so `@zenemic/shared` resolves to shared's built **dist**). Therefore **shared must
be built before the services** — `npm run build` enforces that order.

## Frontend architecture (`main-app/`, Expo)

- Expo SDK **54**, React Native **0.81**, React **19**, TypeScript ~5.9. (The `main-app/README.md`
  still says SDK 51 / RN 0.74 — stale; trust `package.json`.)
- Entry `App.tsx` → `SafeAreaProvider` → `ThemeProvider` (`src/theme.tsx`) → `AppNavigator`
  (`src/navigation/`, React Navigation native stack). Screens in `src/screens/`, reusable UI in
  `src/components/`, the API layer in `src/lib/` (`api.ts`, `supabase.ts`, `auth.tsx`, `config.ts`),
  shared types in `src/data/` + `src/types/`, icons via `react-native-svg` (`src/icons.tsx`).
- Design system tokens (coral `#FF6B4A`, light-default + dark, Inter + JetBrains Mono) all live in
  `src/theme.tsx`. Flow: `Splash → SignUp/Login → Keyboard → Events → EventDetail → PlannerChart`
  plus the create flow `CreateDescribe → CreateConfirm → CreateProcessing → CreateSuccess`.
- The app is **wired to the main-app backend**: Supabase auth (`src/lib/auth.tsx`) gates the navigator
  (logged-out vs logged-in stacks); a typed API client (`src/lib/api.ts`) attaches the Supabase Bearer
  token and parses the backend error envelope; config comes from the repo-root env via `app.config.js`
  → `Constants.expoConfig.extra` (`src/config.ts`). Screens fetch real data (events, AI create, planner
  chart, chat, splitter, settings); optional integrations degrade gracefully on `503 not_configured`.

## The keyboard prototype (`keyboard/`)
HTML/JSX reference (preview via `index.html`): `src/keyboard.jsx` (keys + suggestion bar + prompt
panel), `src/result.jsx` (result sheet + Calendar/Stripe/Map detail views), `src/app.jsx`
(`ZenemicController` state machine `idle → listening → generating → done`). Its backend seams
`callZenemicAPI(prompt)` / `confirmZenemicEvent(event)` in `src/app.jsx` are the integration points
for the `backend/keyboard` service (`/generate`, `/confirm`) — **wiring them up is an intended
future step, not yet done.**

---
> Source: [chromium0308/zenemic](https://github.com/chromium0308/zenemic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
