## ristorantestoria-de

> **IMMER über OpenRouter** (erreichbar via Composio). Nicht Pixelcut/Higsfield für Bild-Edits nutzen.

# ristorantestoria.de

## Bildbearbeitung / Bildgenerierung (projektübergreifend)
**IMMER über OpenRouter** (erreichbar via Composio). Nicht Pixelcut/Higsfield für Bild-Edits nutzen.
Bild-zu-Bild-Editing (z. B. Fotos anpassen) läuft über OpenRouter-Bildmodelle (z. B. Gemini Flash Image / „nano-banana").

## Google Business Profile (GBP) API

**Status:** Aktiv, OAuth eingerichtet (Mai 2026)

**Zugriff via:** `scripts/fetch-google-reviews.mjs`, `scripts/gbp-auth-test.ts`, `scripts/sync-gbp-menu.ts`

**Credentials:**
- OAuth App: "STORIA GBP Sync" (Projekt `storia-gbp-sync`, Owner: sebgruber225@gmail.com)
- Client Secret: `scripts/client_secret.json` (client_id: `62628781665-8kpesh6rpedibo2ipg7sihotkouo1vdv`)
- Tokens: verschlüsselt in Neon DB (`google_business_settings` Tabelle)
- Encryption Key: `GBP_TOKEN_ENCRYPTION_KEY` in `.env`
- Autorisierter Nutzer: `antoine@monot.com` (als Testnutzer eingetragen)

**GBP-IDs:**
- Account ID: `114367954632843728381`
- Location ID: `17586248070861131392`
- Place ID: `ChIJo-_iavt1nkcR665hz1XaMzQ`

**Token erneuern (wenn abgelaufen):**
```bash
npx tsx scripts/gbp-auth-test.ts   # Öffnet Browser auf Port 3000
npx tsx scripts/migrate-tokens-to-db.ts  # Token → Neon DB
```

**Reviews abrufen:**
```bash
node scripts/fetch-google-reviews.mjs --force
```

**Wichtig:** App ist im Test-Modus. Nur `antoine@monot.com` (und sebgruber225@gmail.com) haben Zugriff.
Falls Token abläuft: Re-Auth über `gbp-auth-test.ts` → `migrate-tokens-to-db.ts`.

**Reviews posten/antworten:** Via GBP API v4 (`mybusiness.googleapis.com/v4/accounts/{ACCOUNT_ID}/locations/{LOCATION_ID}/reviews/{REVIEW_ID}/reply`)

**BLOCKING RULE — Rezensionen:** Bei allen Aufgaben rund um Google-Rezensionen (Antworten schreiben, Ton prüfen, Strategie) IMMER zuerst lesen:
`docs/gbp-review-responses.md` — Antwort-Framework, Psychologie, System-Prompt, Verbote

---

## Sub-Agents
| Agent | Model | Zweck |
|-------|-------|-------|
| `architect` | sonnet | Design, Interfaces, Struktur |
| `reviewer` | sonnet | Code-Qualität, Types, Security (read-only) |
| `researcher` | haiku | Codebase-Exploration, Pattern-Suche (read-only) |

Für High-Volume-Output oder Deep Analysis immer Sub-Agent nutzen. `/plan` vor komplexen Features.

## Commands (zusätzlich)
```bash
npm run prerender    # SSG für SEO — KRITISCH
```

## Mehrsprachigkeit (4 Sprachen)
- DE, EN, IT, FR via `src/config/slugs.json` + Translation-Dateien
- hreflang: Alle 4 Sprachen + x-default → de

## SEO URL-Architektur
- Canonical: `https://www.ristorantestoria.de` (mit www, trailing slash IMMER)
- Slugs: `src/config/slugs.json`
- Landing Pages: `/besondere-anlaesse/[slug]/`
- Keyword-Mapping: `docs/seo-strategy.md` (VOR Seitenänderung prüfen!)

## Pre-Render-Regeln (MANDATORY)
- **KEIN `React.lazy()`** für pre-rendered Seiten (nur Admin darf lazy)
- Alle Seiten-Imports in `App.tsx`: eager imports
- Checkliste neue Seite:
  1. Route in `App.tsx` (alle 4 Sprachen via `routeComponents` + `slugs.json`)
  2. Slug in `src/config/slugs.json` (de, en, it, fr)
  3. Slug in Translation-Dateien (`de.ts`, `en.ts`, `it.ts`, `fr.ts` → `slugs`-Objekt)
  4. Eager import (kein `lazy()`)
  5. `npm run build` ok (105+ Routen, 0 Errors)
  6. HTML in `dist/` hat echten Content (kein "Laden...")
  7. `<title>`, `<meta description>`, hreflang, JSON-LD vorhanden
  8. Translations in allen 4 Sprachen
- Verify: `find dist -name "index.html" -exec grep -l "Laden\.\.\." {} \;` → muss leer sein

## SEO Content Rendering
- Accordion: IMMER `forceMount` + `data-[state=closed]:hidden`
- Dynamic Content: SSG-prerendered, nicht client-only
- Video: `preload="none"`

## Rechtliche Seiten (NUR Deutsch)
- impressum, datenschutz, cookie-richtlinie, agb-restaurant, agb-gutscheine, widerrufsbelehrung, zahlungsinformationen, lebensmittelhinweise, haftungsausschluss
- `noHreflang` im `<SEO>`-Tag
- `LEGAL_ONLY_DE` konsistent in: `routes.ts`, `App.tsx`, `generate-sitemap.mjs`
- `.htaccess`: 301-Redirects alter übersetzter URLs → DE

## Local SEO
- NAP: Ristorante STORIA, Karlstraße 47a, 80333 München
- Tel: +49 89 51519696
- GMB: Italian Restaurant (Primary), Pizza Restaurant, Wine Bar
- Keywords: "italienisches restaurant münchen", "pizza münchen maxvorstadt", "neapolitanische pizza münchen"

## Content-Architektur
### Pillar Pages
| Pillar | URL |
|--------|-----|
| Besondere Anlässe | `/besondere-anlaesse/` |
| Speisekarte | `/speisekarte/` |
| Über Uns | `/ueber-uns/` |

### Landing Pages
lunch-muenchen-maxvorstadt, aperitivo-muenchen, romantisches-dinner-muenchen, eventlocation-muenchen-maxvorstadt, firmenfeier-muenchen, geburtstagsfeier-muenchen, neapolitanische-pizza-muenchen, wild-essen-muenchen

## SEO Indexing Tool

Google Indexing API Script zum Einreichen von URLs zur Indexierung (200/Tag kostenlos).
Service Account JSON liegt in `scripts/service-account.json` im seo.schrittmacher.ai Repo.

```bash
# Optionen
node scripts/request-indexing.mjs --de-only        # Nur DE-URLs aus Sitemap
node scripts/request-indexing.mjs --priority        # Nicht-indexierte Priority-URLs
node scripts/request-indexing.mjs --dry-run         # Preview ohne Submit
node scripts/request-indexing.mjs --dry-run "https://www.ristorantestoria.de/<pfad>/"  # Einzelne URL (positional), Preview
node scripts/request-indexing.mjs "https://www.ristorantestoria.de/<pfad>/"            # Einzelne URL (positional), Submit
```

**Service Account:** `gsc-auditor-storia@evocative-shore-486623-v4.iam.gserviceaccount.com`
Muss in GSC als **Inhaber** für `www.ristorantestoria.de` eingetragen sein.

---
> Source: [dream-anchor/ristorantestoria.de](https://github.com/dream-anchor/ristorantestoria.de) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
