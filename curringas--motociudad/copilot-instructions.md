## motociudad

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Full instructions, workflow rules, and subagent specs live in `docs/CLAUDE.md` and `docs/AGENTS.md`. Read those before starting non-trivial work.

---

## Project overview

**MotoCiudad** is a collaborative mobile app for finding, proposing, and verifying motorcycle parking spots, sustained by a gamified community (Octanos points, levels, badges). Currently in **Spec Driven Development** MVP phase — all architecture decisions are documented in `docs/`.

---

## Stack

```
Mobile      : React Native + Expo SDK 52+ · TypeScript strict · Expo Router v4
State       : Zustand (client) + TanStack Query v5 (server)
Styling     : NativeWind 4 (Tailwind for RN)
Backend     : Supabase Cloud — PostgreSQL 15 + PostGIS + Auth + Storage + Edge Functions
Edge Funcs  : Deno + TypeScript
Validation  : Zod (client and edge)
Tests       : Vitest, RN Testing Library, Maestro (E2E), pgTAP (RLS)
CI/CD       : GitHub Actions + EAS Build + EAS Update (OTA)
```

---

## Commands

```bash
# Setup
pnpm install
supabase start              # starts local DB in Docker

# Development
pnpm dev:mobile             # Expo dev server
pnpm typecheck              # tsc --noEmit across all packages
pnpm gen:types              # regenerate TS types from Supabase schema

# Tests
pnpm test                   # Vitest unit + integration
pnpm test --coverage
maestro test .maestro/      # E2E (requires simulator/device)
supabase test db            # pgTAP (RLS + SQL functions)
deno test supabase/functions/**/*.test.ts

# Database
supabase migration new <description>
supabase db push
supabase db reset

# Build & deploy
eas build --platform all --profile preview
eas submit --platform all --profile production
eas update --branch production --message "<msg>"
```

---

## Architecture

The app has two main layers:

**Mobile app** (`apps/mobile/`) — React Native with Expo Router file-based routing. Features are organized as vertical slices under `features/<domain>/` each containing `api.ts` (Supabase calls), `hooks.ts` (TanStack Query), `schemas.ts` (Zod), and `components/`.

**Supabase backend** (`supabase/`) — PostgreSQL with PostGIS for geospatial queries. Authorization is enforced exclusively via Row Level Security policies — no API gateway. Business logic with side effects (awarding Octanos, badge checks, ranking computation) lives in Deno Edge Functions, never in client code.

Key data flow: client → Edge Function (auth + validation) → RLS-protected PostgreSQL → real-time subscriptions back to client.

---

## Canonical docs

Before modifying any area, read the relevant doc:

| Doc | When to consult |
|-----|----------------|
| `docs/prd.md` | Before any new feature |
| `docs/arquitectura.md` | Before introducing new libraries or patterns |
| `docs/modelo-datos.md` | Before migrations or query changes |
| `docs/gamificacion.md` | Before touching Octanos / badges / levels |
| `docs/testing.md` | Before adding or modifying tests |
| `docs/infraestructura.md` | Before changing CI/CD or secrets |

If code and specs diverge, **update the spec in the same PR** (or delegate to the `prd-keeper` subagent). The repo must never be in a state where code and specs contradict each other without documented justification.

---

## Non-negotiable rules

1. Never touch `octano_events` directly from the client — always via Edge Function.
2. Never expose `service_role_key` in client code.
3. Never persist user geolocation except the `user_location` field in a specific verification (privacy rule).
4. Every new table requires RLS enabled + at least one policy + a pgTAP test in the same PR.
5. Never `DROP` a column in a migration without a deprecation release first.
6. Do not implement features absent from `docs/prd.md` without explicit confirmation.
7. **Verificación de cierre de `opsx:apply`**: ningún change se considera aplicado —ni se archiva— hasta ejecutar el skill `verify-all-platforms` (vía subagente `e2e-verifier`) como último paso: E2E en las plataformas donde la feature se ve (app móvil → web + Android + iOS; panel admin → solo web), logueado como usuario y como admin donde aplique, con limpieza de datos de prueba. Debe dejar evidencia en `.claude/verify-runs/<change>.md` (un hook bloquea `openspec archive` sin ella).

---

## Conventions

- **Language**: code and comments in English; UI copy, docs, and commit messages in Spanish (es-ES).
- **TypeScript**: `strict: true`, `noUncheckedIndexedAccess: true`, absolute imports via `@/`.
- **Commits**: Conventional Commits (`feat(domain):`, `fix(domain):`, `test(domain):`, etc.).
- **SQL**: tables plural/snake_case; one atomic idea per migration; no `SELECT *` in production.
- **Components**: PascalCase filenames, one component per file; hooks prefixed `use`.

---

## Out of scope for MVP

- Light theme
- Private messaging or forums
- Payments or reservations
- Multi-language (es-ES only; i18n-ready but single locale)
- Bulk data import from external sources

---
> Source: [curringas/motociudad](https://github.com/curringas/motociudad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
