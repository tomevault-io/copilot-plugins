## vodni-stav-cz

> Tento soubor poskytuje kontext pro Claude Code (claude.ai/code) při práci s tímto repozitářem.

# CLAUDE.md

Tento soubor poskytuje kontext pro Claude Code (claude.ai/code) při práci s tímto repozitářem.

## Komunikace

- Komunikuj česky a tykej uživateli (Martin).

## Přehled projektu

**vodnistav.cz** — agregátor hydrologických dat z ČHMÚ. Stahuje data o vodních stavech a průtocích z otevřených API ČHMÚ do PostgreSQL, zobrazuje přes Next.js aplikaci.

## Příkazy

- `yarn dev` — vývojový server Next.js
- `yarn build` — produkční build
- `yarn lint` — ESLint
- `yarn migrate` — spuštění databázových migrací (advisory locks)
- `yarn ingest` — ruční spuštění ingestu dat (`tsx scripts/ingest/index.ts`)

## Architektura

**Stack:** Next.js 16 (App Router), React 19, PostgreSQL, raw SQL přes `postgres` (Postgres.js), Material UI (MUI), Recharts, deploy na Hetzner VPS (standalone build + Caddy reverse proxy).

**Bez Tailwind** — UI je čistě přes MUI (`sx` prop, MUI komponenty). Nepoužívat Tailwind utility třídy.

**Package manager:** Yarn (classic) s `node_modules` strategií. Vždy používej `yarn` místo `npm`.

**Bez ORM** — veškerý přístup k databázi přes tagged template SQL dotazy skrz singleton `postgres()` instanci (`src/lib/db.ts`, cachovaná na `global.__sql`).

### Tok dat

```
ČHMÚ open data API → scripts/ingest/chmi.ts → PostgreSQL → Next.js frontend
```

Ingest běží denně ve 23:55 UTC přes systemd timer (`vodnistav-daily-ingest.timer`). Dvě fáze:

1. **Discovery** — stáhne metadata stanic z ČHMÚ meta1.json, upsertne povodí/řeky/stanice
2. **Ingest měření** — paralelní fetch JSON souborů per stanice, sloučení H (vodní stav cm) a Q (průtok m³/s) časových řad, batch upsert s řešením konfliktů

### Klíčové soubory

- `scripts/ingest/chmi.ts` — logika stahování a parsování dat z ČHMÚ
- `scripts/ingest/extractors.ts` — parsování meta1 JSONu
- `scripts/ingest/utils.ts` — DB helpery (audit ingest runů)
- `scripts/cron/daily-ingest.ts` — standalone denní ingest (systemd timer)
- `scripts/cron/historical-ingest.ts` — historický backfill (systemd timer)
- `scripts/cron/subscription-notifier.ts` — emailové notifikace (systemd timer)
- `src/lib/db.ts` — singleton PostgreSQL pool
- `migrations/001_init.sql` — schéma: basin, river, station, measurement, ingest_run

### Deploy (Hetzner VPS)

- `deploy/vodnistav.service` — systemd service pro Next.js aplikaci
- `deploy/vodnistav-*.timer` — systemd timery pro cron joby
- `deploy/Caddyfile` — reverse proxy s automatickým HTTPS
- `.github/workflows/deploy.yml` — CI/CD pipeline (lint, typecheck, test → SSH deploy)

### Databázové schéma

Hlavní tabulky: `basin → river → station → measurement` (hierarchické). Měření mají kompozitní PK `(station_id, ts)` pro přirozenou deduplikaci. `ingest_run` slouží jako audit trail. Všechny tabulky používají INTEGER identity sloupce a JSONB pro flexibilní metadata.

## Styl kódu

- **Pouze arrow funkce** — `func-style: expression` vynuceno, žádné `function` deklarace
- **Arrow callbacks** — `prefer-arrow-callback: error`
- **Vyhýbat se `any`** — `@typescript-eslint/no-explicit-any: warn`
- **Formátování** (Prettier): jednoduché uvozovky, středníky, trailing čárky, šířka 100 znaků, 2 mezery odsazení, LF konce řádků
- **Path alias:** `@/*` mapuje na kořen projektu

## Lokální databáze

Dev prostředí používá lokální PostgreSQL 17 (`postgres://martinzadrazil@localhost:5432/hydro`). Produkce běží na PostgreSQL na Hetzner VPS.

**Migrace na lokální DB lze pouštět bez ptaní** — `yarn migrate` nebo `DATABASE_URL="postgres://martinzadrazil@localhost:5432/hydro" yarn migrate`.

## Proměnné prostředí

Viz `.env.example`. Povinné: `DATABASE_URL`, `CHMI_META1`, `CHMI_NOW_INDEX`, `CRON_SECRET`. Volitelné: `ONLY_STATIONS` (čárkami oddělený filtr), `FETCH_DELAY_MS`.

## Workflow pravidla

Po každé změně kódu:

1. Spusť `yarn lint` a oprav všechny chyby
2. Spusť `yarn typecheck` a oprav TypeScript chyby
3. Ke každému přidání nebo změně logiky se snaž dopsat testy
4. Pokud existují testy pro změněnou oblast, spusť je
5. Po vytvoření migrace ji automaticky spusť nad localhost databází: `yarn migrate`
6. Považuj úkol za hotový až když všechny předchozí kroky projdou čistě.

---
> Source: [claryaldringen/vodni-stav.cz](https://github.com/claryaldringen/vodni-stav.cz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
