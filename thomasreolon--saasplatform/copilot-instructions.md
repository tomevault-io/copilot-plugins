## saasplatform

> Blueprint repo per SaaS basati su crediti e job asincroni. Focus sulla core feature di valore, non sull'infrastruttura.

# Job Based SaaS Blueprint

## Why
Blueprint repo per SaaS basati su crediti e job asincroni. Focus sulla core feature di valore, non sull'infrastruttura.
Stato attuale: side project dev-only. Codice billing commentato; crediti gratuiti: 100 al signup + 1 ricarica gratuita da 100 (l'admin `ADMIN_EMAIL` ricarica senza limiti). Anti multi-account: hash SHA-256 (peppered, `IP_HASH_SALT`) dell'IP di signup nel registro append-only `signup_ip_seen` — IP già visto = niente bonus né ricarica, e sopravvive alla cancellazione GDPR.

## What
**Stack:**
- **Landing**: Astro (zero-JS, SEO-first) — app standalone, servita separatamente (CDN in prod).
- **App**: React SPA (`apps/app`, react-router + TanStack Query). Buildata e **servita dal backend (monolite)** alla root della sua origin (dev: `:8000`; prod: `app.domain.com`), same-origin con l'API (niente CORS). In dev gira su Vite (`:3000`) con proxy `/api` → `:8000`.
- **Backend**: FastAPI (Python) — espone `/api/v1/*` e serve la SPA compilata da `static/app` (catch-all sulla root per il client routing; prefissi `api`/`health`/`openapi.json` riservati).
- **Auth**: Firebase Auth (ID Bearer token verified server-side via Admin SDK).
- **DB**: Postgres, SQLModel/SQLAlchemy, Alembic.
- **Job queue**: Postgres-as-queue (`SELECT FOR UPDATE SKIP LOCKED`).
- **File storage**: Google Cloud Storage per i file di input/output dei job (opzionale per job). In dev fake-gcs-server (stesso code path). I byte stanno su GCS, i metadati su Postgres (`artifacts`).
- **Billing**: Stripe (scaffolding commentato/non attivo, crediti gratis).

**Struttura:**
```
.
├── apps/
│   ├── web/              # Astro (landing, standalone)
│   ├── app/              # React SPA (servita dal backend sotto /app)
│   └── api/              # FastAPI (API /api/v1/* + serve la SPA → monolite)
├── jobs/                 # Worker Python (Cloud Run Jobs)
├── libs/shared/          # Core condiviso (db, billing, storage, catalog dei job)
│   └── src/shared/catalog/   # ← un file per job (definizione completa)
├── infra/                # Docker compose / config infra
└── CLAUDE.md
```

**Modello dati core:** `users`, `credit_transactions` (ledger append-only), `jobs` (stato + output), `artifacts` (file di input/output, byte su GCS), `audit_log`. **Il saldo crediti è sempre `SUM(credit_transactions.amount) WHERE user_id = ?` (mai cachato).**

**Componenti React riusabili (`apps/app/src/components/`):**
- `ArtifactPreview.tsx` — visualizza artefatti di output (immagini, file) di un job completato
- `FileField.tsx` — input per upload di file con gestione async e barra di progresso
- `Icons.tsx` — set di icone SVG inline (usa solo queste; non aggiungere librerie esterne)
- `Marketplace.tsx` — modal per abilitare/disabilitare gruppi di job
- `Notifications.tsx` — dropdown notifiche nella topbar
- `Topbar.tsx` — barra di navigazione superiore (breadcrumb, tema, lingua, notifiche)
Prima di creare un nuovo componente verifica sempre se ne esiste già uno riusabile qui.
Pagine principali: `JobsPage.tsx` (lista job + 3 tab per tipo), `JobView.tsx` (form nuova esecuzione), `SettingsPage.tsx`, `Sidebar.tsx`, `Login.tsx`.

**App SPA — cartelle e pattern (`apps/app/src/`):** la logica condivisa vive in `lib/` (una sola via per ogni cosa):
- `api.ts` — UNICO wrapper fetch (bearer token + gestione errori). Mai `fetch` grezzo o axios.
- `queries.ts` — hook TanStack Query (`useJobs`, `useJobTypes`, …): unico modo per leggere/mutare dati server.
- `types.ts` — specchio TS delle response API; tienilo allineato ai campi usati.
- `i18n.ts` — `t(key)` per le stringhe statiche dell'UI (mai hardcodare testo in JSX), `td(str)` per le stringhe dinamiche dal catalog (display_name/description/titoli di output).
- `groups.ts` — raggruppa i job per `group` e annida i `subgroup` (es. marketing > reels) nella Sidebar.
Form di input e pagina risultati sono **generati dallo schema**: non scrivere UI per-job. Lo stile sta in `styles/global.css` (design token + classi, no CSS-in-JS).

**Aggiungere un job: UN solo file.** Crea `libs/shared/src/shared/catalog/<tipo>.py` (copia da `_template.py` nella stessa cartella). Tutto sta lì dentro:
- **Schemi** input/output (Pydantic). Per input-file usa `file_field(accept=..., max_mb=..., multiple=...)`; per gli output usa `result_field(artifact=..., format="eur|duration|number|text", hide=...)` così la pagina risultati mostra stat curate invece del JSON (gli artifact PDF/video/immagine sono resi inline).
- **Metadati + handler** via il decoratore `@job(type=..., cost=..., group=..., subgroup=..., name=..., input=..., output=...)` sulla funzione `run(ctx, inp) -> Output` (`subgroup` opzionale = sotto-cartella in sidebar).
- **Costo in crediti**: `cost=usd(<costo vivo stimato in $>)` da `shared.pricing` (1 credito ≈ 1¢ venduto, copre $0.006 di costo → margine 40%; minimo 10 crediti/job) con un commento che documenta cosa pesa. Se il costo scala con l'input (es. pagine di un PDF): `cost_per_unit=usd_per_unit(...)` + `cost_unit={"it":...,"en":...}` + `count_units=<fn(inp, load)>` — le unità vengono misurate al submit, prima dell'addebito (vedi `pdf_translate.py`).

Auto-discovery: il package `shared.catalog` importa da solo ogni file (esclusi quelli con prefisso `_`) e popola il registry, visto identico da API e worker. Niente `catalog.yaml`, niente registrazione manuale. I file: `ctx.input_path(inp.campo)` per leggere input, `ctx.save_output(path)` per produrre output; le dipendenze pesanti (es. Pillow) vanno importate **dentro `run()`** così l'API resta leggera. La validazione di payload e file è automatica lato API; form e render degli output sono generati dallo schema. `jobs/tests/test_registry.py` verifica che ogni job sia completo.

**Da sapere quando sviluppi (gotcha che rompono in silenzio):**
- **Import pesanti SOLO dentro `run()`** e **moduli di supporto col prefisso `_`** (es. `_reels/`): altrimenti l'API si appesantisce o la discovery prova a registrarli come job e fallisce.
- **Logica di dominio condivisa** → in un package `_<famiglia>/` (vedi `_reels/`, `_leadgen/`); il file del job resta sottile (schemi + `run()`).
- **Hot-reload dev**: API e catalog ricaricano da soli; il **worker no** → `docker compose restart worker` dopo aver toccato un job. `useJobTypes` ha `staleTime: Infinity` → dopo modifiche al catalog **hard-refresh** del browser.
- **Frontend, una sola via**: dati server solo via `lib/queries.ts` (mai `fetch` grezzo: usa `lib/api.ts`); nessuna stringa hardcoded in JSX (`t()`/`td()`); form e pagina risultati **generati dallo schema** (`file_field`/`result_field`) — non scrivere UI per-job.
- **UI**: riusa le primitive in `components/ui.tsx` (`Button`/`Card`/`Input`) e le classi/token di `global.css`; controlla `components/` prima di crearne di nuove. Icone solo da `Icons.tsx`.
- **Vincoli**: schema DB solo via migrazioni Alembic; crediti consumati in transazione atomica; mai committare segreti né `node_modules` nel build context (c'è `.dockerignore`).

## How
**Setup:**
1. `cp .env.example .env` e configura.
2. `docker compose up --build` (prima volta, poi basta `docker compose up`).
3. Landing: `http://localhost:4321` | App: `http://localhost:8000` | SPA dev: `http://localhost:3000`

**Dev workflow (hot-reload, no rebuild):**
`docker-compose.override.yml` è caricato automaticamente da `docker compose up` e monta le sorgenti come volumi:
- API Python (`apps/api/app/`, `libs/shared/`) → `uvicorn --reload` ricarica ad ogni salvataggio `.py`
- Catalog job (`libs/shared/src/shared/catalog/`) → live nell'API; `docker compose restart worker` per il worker
- SPA React → Vite HMR già incluso nel compose (`frontend` service) → `http://localhost:3000`; oppure `pnpm --filter app dev` localmente come alternativa
- Per fare girare il solo stack di supporto (db, storage, worker) senza l'app: `docker compose up db storage worker`

**Prod build (salta l'override):**
`docker compose -f docker-compose.yml up --build`

**Comandi:**
- **Stack**: `docker compose up` | `down` | `down -v` (reset DB)
- **Migrations**: `uv run alembic revision --autogenerate -m "msg"` | `uv run alembic upgrade head`
- **Quality**: `pnpm verify` (tutto) | `pnpm test && uv run pytest` (test) | `pnpm typecheck && uv run mypy apps/api` (types) | `pnpm lint --fix && uv run ruff check --fix` (lint)

**Env & Lingua**: Unico `.env` nella root. Lingua: commenti in italiano, codice/logs/commit in inglese.

## Regole non negoziabili
- **No Secrets**: Nessuna credenziale/chiave committata. `.env` in `.gitignore`.
- **Firebase Auth**: ID token Bearer verificato lato server, mai fidarsi del client.
- **Transazioni Crediti**: Consumo atomico in transazione DB (`SELECT FOR UPDATE` su utente + check saldo `SUM(...)` + insert transaction).
- **DB & Migrations**: Schema modificabile solo tramite migrazioni Alembic. No `create_all()`.
- **No logs grezzi**: Niente `console.log` o `print` in prod. Usa logger strutturato.
- **Pre-commit & CI**: Hook locali e CI bloccanti per linting/formatting.
- **GDPR**: Endpoint per export e cancellazione account obbligatori.
- **Stripe**: Codice di billing commentato e inattivo fino a decisione contraria.

## Guideline Design & UX
- **Estetica Premium (WOW factor)**: Design moderno (glassmorphic, dark-mode sleek, palette curate come HSL o tinte scure armoniose, tipografia elegante es. Inter/Outfit, no colori default o grigi spenti, gradienti morbidi e micro-animazioni).
- **Stato Dinamico**: Hover e focus evidenti, transizioni fluide su bottoni e card. No placeholder brutti: usa immagini reali o generatori.
- **UX Robusta**: Skeleton loader sopra il fold (no jumps), form con validazione inline, submit disabled durante l'invio, error boundaries per route (mai schermo bianco).
- **Mobile-first/Responsive**: Desktop-first per app complesse ma adattabile su mobile senza rotture (min 320px).
- **i18n + Empty States**: Lingue IT+EN da subito (no stringhe hardcoded in JSX). Empty states curati con CTA chiara.

## Guideline Codebase
- **KISS & Dry**: Codice minimale, funzioni <50 righe, file <300 righe, astrazioni solo dopo 3 usi reali.
- **One Canonical Way**: Un solo pattern per ogni problema (fetch, handling, routing).
- **Colocation & Custom Hooks**: Stato/logica vicini ai componenti. Custom hooks per logica complessa (>10 righe).
- **Composition**: Componenti piccoli, componibili, preferendo `children` a config-object complessi.
- **Strict Typing**: TS strict nel frontend, type hints + mypy in backend, tipi condivisi via OpenAPI.

## Guideline Performance / SEO
- **Core Web Vitals**: LCP <2.5s, CLS <0.1, INP <200ms. Lazy-load below-the-fold (no LCP).
- **Astro Shell**: Minimal JS, `client:visible` / `client:idle` di default, `client:load` solo se interattivo subito.
- **SEO & Meta**: Meta dinamici, JSON-LD, sitemap/robots auto, URL semantici (no query params per ID).

## Guideline Backend / Sicurezza
- **Security & CORS**: HTTPS, HSTS, CORS strict (no `*`), rate limiting (SlowAPI), API versionate `/api/v1/`.
- **Validazione & Idempotenza**: Pydantic per ogni input, Idempotency keys su POST distruttivi.
- **Logs & Audit**: Structured log JSON con `request_id`, audit log append-only per azioni critiche (who/what/when/IP).
- **SOC2-ready**: Codice robusto (least privilege, backup testati), ma nessuna certificazione formale per ora.

---
> Source: [thomasreolon/SaaSPlatform](https://github.com/thomasreolon/SaaSPlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
