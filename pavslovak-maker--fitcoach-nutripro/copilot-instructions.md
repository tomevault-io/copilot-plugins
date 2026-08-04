## fitcoach-nutripro

> Full-stack fitness coaching app. Backend: Fastify + Prisma + PostgreSQL + Redis (TypeScript). Frontend: Next.js PWA (PowerShell / Windows dev).

# FitCoach & NutriPro — kontext projektu

## Co je to

Full-stack fitness coaching app. Backend: Fastify + Prisma + PostgreSQL + Redis (TypeScript). Frontend: Next.js PWA (PowerShell / Windows dev).

## Stack

| Vrstva | Tech |
|--------|------|
| API | Fastify, Prisma, PostgreSQL, BullMQ (Redis) |
| AI | Anthropic API (`claude-sonnet-4-6`) — 2 agenti: FitCoach + NutriPro |
| Frontend | Next.js (App Router), Tailwind, PWA |
| Validace | Zod |
| Testy | Vitest |

## Klíčové soubory

```
src/
  ai/
    fitcoach-agent.ts     # AI agent — tréninkový plán (7 dní)
    nutripro-agent.ts     # AI agent — nutriční plán (trénink + odpočinek)
    orchestrator.ts       # FitCoach → NutriPro → cross-check
    cross-check.ts        # 9 bezpečnostních pravidel (kód, ne AI prompt)
  services/
    ai-service.ts         # generateAndSavePlan() — hlavní flow
  validators/
    schemas.ts            # Zod schémata (API vstupy + AI výstupy)
  api/
    routes.ts             # /today, /ai/generate a další endpointy
  types/
    domain.ts             # TypeScript typy
web/app/(app)/
  page.tsx                # Home screen — zobrazuje dnešní plán
  profile/page.tsx        # Tlačítko "Vygenerovat nový plán"
prisma/schema.prisma      # DB schema (DayOfWeek enum: po|ut|st|ct|pa|so|ne)
```

## Co jsme řešili

### 1. calorie_change_limit: 10% → 20%
- `src/ai/cross-check.ts` — `checkCalorieChangeLimit()` (Rule 1)
- `tests/cross-check.test.ts` — testy aktualizovány + přidán test pro 15% (mezi 10–20%)
- `docs/ARCHITECTURE.md` — tabulka pravidel aktualizována
- **Pozor:** system prompt NutriPro (`nutripro-agent.ts` řádek 24) stále říká "max 10%" — je to záměrně konzervativní instrukce pro AI, cross-check ale pustí až 20%.

### 2. Bug: Home screen nezobrazoval vygenerovaný plán
**Root cause:** FitCoach a NutriPro parsovaly AI odpověď jen přes `JSON.parse()` bez validace. AI mohla vynechat den v týdnu nebo typ dne (tréninkový/odpočinkový) — uložilo se jako "úspěch", ale `/today` endpoint nenašel data.

**Oprava:**
- `src/validators/schemas.ts` — přidány `FitCoachOutputSchema` + `NutriProOutputSchema` (Zod)
  - FitCoach musí vrátit všech 7 dní (`po|ut|st|ct|pa|so|ne`)
  - NutriPro musí vrátit jídla pro oba typy dní (`je_treninkovy_den: true` i `false`)
- `src/ai/fitcoach-agent.ts` — `.safeParse()` po `JSON.parse()`, hází chybu s popisem co chybí
- `src/ai/nutripro-agent.ts` — stejný pattern

## Jak spustit (PowerShell / Windows)

```powershell
# Backend (root složka)
docker compose up -d postgres redis
npm run db:migrate
npm run dev                    # API na :3001

# Frontend (nové okno)
cd web
npm run dev                    # Next.js na :3000

# Testy
npm test
```

**.env** (do kořene projektu):
```
DATABASE_URL=postgresql://fitcoach:fitcoach_dev@localhost:5432/fitcoach_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=cokoliv
ANTHROPIC_API_KEY=sk-ant-...
```

## Známé pre-existující TS chyby (netýkají se našich změn)

- `fitcoach-agent.ts(171)` + `nutripro-agent.ts(176)` — `response.json()` vrací `unknown`, přistupujeme na `.content`
- `server.ts(76,77,83,85)` — error handler přistupuje na `.name/.message/.statusCode` na `unknown`

## DayOfWeek enum

`po | ut | st | ct | pa | so | ne` — musí sedět s dnešním dnem při dotazu na `/today`.

---
> Source: [pavslovak-maker/fitcoach-nutripro](https://github.com/pavslovak-maker/fitcoach-nutripro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
