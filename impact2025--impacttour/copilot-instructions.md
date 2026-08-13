## impacttour

> Next.js 15 (App Router) platform waar teams een GPS-route lopen, checkpoints

# IctusGo — GPS Teambuilding Platform

Next.js 15 (App Router) platform waar teams een GPS-route lopen, checkpoints
ontgrendelen via locatie, missies voltooien en door AI worden gescoord op vier
**GMS-dimensies**: verbinding, betekenis, plezier, groei (elk 0–25 per checkpoint).

> Productnaam is **IctusGo**. De package/repo heet historisch `impacttour` en het
> Vercel-domein is `impacttour.vercel.app` — gebruik in nieuwe, gebruiker-zichtbare
> tekst altijd "IctusGo".

## Stack

| Laag | Keuze |
|------|-------|
| Framework | Next.js 15 App Router, React 19 |
| DB | Drizzle ORM + Neon Postgres (`src/lib/db/schema.ts`) |
| Auth | NextAuth v5 (beta) — **alleen magic link**; wachtwoord-login via server actions (zie onder) |
| Realtime | Pusher (`src/lib/pusher`) |
| Betalingen | Stripe (`/api/stripe/*`) |
| AI | OpenRouter → Claude Sonnet 4.6 (zwaar) + Haiku (snel) via `src/lib/ai` |
| Opslag | Vercel Blob (foto's) |
| Rate limiting | Upstash Redis + in-memory fallback (`src/lib/rate-limit.ts`) |

## Commando's

- `bun dev` — dev server op poort **7080**
- `bun run build` / `bun start`
- `bun run db:generate` / `db:migrate` / `db:push` / `db:studio` (Drizzle)
- Typecheck: `./node_modules/.bin/tsc --noEmit -p tsconfig.json`

## Auth — belangrijk

NextAuth v5 (beta) maakt met `strategy: 'database'` **geen sessie aan voor
Credentials-logins**. Daarom:

- **Magic link** (spelleider) loopt via NextAuth Resend-provider (`src/lib/auth/index.ts`).
- **Wachtwoord-login** (admin, spelleider, klant) loopt via **server actions** die
  zelf een DB-sessie aanmaken met `createUserSession()` (`src/lib/auth/session.ts`):
  - `src/app/(admin-auth)/admin/login/actions.ts`
  - `src/app/login/actions.ts`
- Beide login-actions zijn rate-limited via `checkRateLimit` (Redis/in-memory).
- Middleware (`src/middleware.ts`) checkt alleen het bestaan van de sessie-cookie;
  rol-verificatie gebeurt in layouts/pages via `auth()`.

Voeg **geen** Credentials-provider toe aan de NextAuth-config — die werkt niet met
de database-session strategie. Gebruik het server-action + `createUserSession`-patroon.

## AI-laag (`src/lib/ai/index.ts`)

Functies A–G: opdracht-generatie, hints (3 niveaus), antwoord-evaluatie + GMS-scoring,
coach-inzicht, debriefing, tocht-generator, impactrapport-narratief.

- `DEFAULT_MODEL` (zwaar) voor narratieven/JSON; `FAST_MODEL` (Haiku) voor hoog-volume
  gameplay-calls — 10–15× goedkoper.
- `aiCompleteJSON` parseert robuust via `extractJson` (code fences + omringende proza)
  en valt bij modelfout terug op `DEFAULT_MODEL`.
- Bij AI-uitval valt scoring terug op **proportionele basisscore** (60% van max) —
  gameplay blijft werken zonder AI. Zie `/api/game/submit`.
- Model-ID's via OpenRouter **zonder** datum-suffix. Let op: bij upgrade naar
  Opus 4.7/4.8 wordt `temperature` afgewezen (400) — nu Sonnet 4.6 dus prima.

## Game-API conventies (`/api/game/*`)

State-wijzigende routes gebruiken consistent:
- `checkOrigin(req)` — CSRF-bescherming (eerst, vóór parsing)
- `checkRateLimit(...)` waar relevant
- Zod-schema voor body-validatie
- Team-auth via `teamToken` (geen individuele namen opgeslagen voor kids-varianten)
- Server-side GPS-validatie via Haversine in `/api/game/checkpoint` (anti-cheat)

## Privacy / AVG

Kids-varianten (`jeugdtocht`, `voetbalmissie`): geen individuele namen (alleen
`teamToken`), foto's krijgen `scheduledDeleteAt` (+30 dagen), ouderlijke toestemming
als sessie-veld. Chat is uitgeschakeld voor kids-varianten (alleen hint-knoppen).

## Tour-varianten

`wijktocht` · `impactsprint` · `familietocht` · `jeugdtocht` · `voetbalmissie`
(zie `tourVariantEnum` in het schema).

---
> Source: [Impact2025/impacttour](https://github.com/Impact2025/impacttour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
