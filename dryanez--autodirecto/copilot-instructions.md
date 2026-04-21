## autodirecto

> > This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

# Agent Instructions for Autodirecto

> This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

> **📖 REQUIRED READING: [`PROJECT_OVERVIEW.md`](./PROJECT_OVERVIEW.md)** — This is your primary knowledge base. It explains the full architecture, every folder, every system (SimplyAPI, MrCar, Funnels, Camera PWA, ChileAutos), all API routes, database tables, and the complete business flow. **Read it before diving into any code work.**

## What is Autodirecto?

**Autodirecto** is a car consignment platform for Chile. We help people sell their cars by:
1. AI-powered valuation (MrCar engine)
2. Professional photography (Camera PWA with ghost overlays)
3. Digital contract signing
4. Multi-channel publishing (autodirecto.cl + ChileAutos.cl)
5. CRM pipeline management (leads → consignment → inspection → sale)

**Core Systems:**
- **Next.js Frontend** (autodirecto.vercel.app) — Public site, catalog, vehicle detail pages
- **SimplyAPI** (autodirectocrm.vercel.app) — Flask backend + Alpine.js CRM dashboard (THE BRAIN)
- **Camera PWA** (cameracar.vercel.app) — Mobile photo capture with AR-like overlays
- **MrCar** (mrcar-cotizacion.vercel.app) — AI pricing engine for Chilean market
- **ChileAutos Integration** — Publish to Chile's largest marketplace + receive buyer leads
- **Supabase** — PostgreSQL database + storage for all data and photos

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- Basically just SOPs written in Markdown, live in `directives/`
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `directives/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**
- Deterministic Python scripts in `execution/`
- Environment variables, api tokens, etc are stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of manual work. Commented well.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Key Context for Autodirecto

**Database & Data Flow:**
- All data lives in **Supabase (PostgreSQL)** at `kqympdxeszdyppbhtzbm.supabase.co`
- `db.py` is a compatibility layer that translates SQL-like queries to Supabase REST API
- **Bidirectional sync**: Changes to owner info in CRM leads auto-sync to consignaciones and vice versa
- Matching logic: Supabase ID → plate → RUT → phone

**Critical Tables:**
- `consignaciones` — Core entity (consigned vehicles) with ChileAutos integration fields
- `listings` — Published catalog entries with `chileautos_id`, `chileautos_status`
- `crm_leads` — All leads with `ai_consignacion_price` (saved, never recalculated!)
- `compradores` — Buyers with `source` field (manual/chileautos)
- `appraisals` — Vehicle inspections
- `camera_jobs` — Photo session tokens for Camera PWA
- `crm_settings` — Key-value config (WhatsApp, ChileAutos creds)

**Business Logic Rules:**
1. **AI pricing is cached**: When "Tasar con IA" runs in Funnels, price saves to `crm_leads.ai_consignacion_price`. Wizard reuses this—NO recalculation!
2. **Auto-unpublish**: When consignación status → `vendida` or `cancelada`, automatically unpublishes from ChileAutos
3. **ChileAutos creds in DB**: Stored in `crm_settings` table (not env vars) for UI configurability
4. **Token-based camera auth**: No login—CRM generates unique token → inspector opens URL → photos upload to correct Supabase folder

**API Architecture:**
- Next.js API routes (`/api/*`) are **proxies only**—no business logic except `normalizeRow()` in listings
- SimplyAPI `app.py` (~5,500 lines) contains ALL business logic
- Camera PWA uploads directly to Supabase Storage (bypasses backend)

**Where Things Live:**
- `src/app/` — Next.js pages (landing, catalog, wizard, detail pages)
- `src/app/api/` — Next.js API proxies
- `src/app/components/` — React components
- `SimplyAPI/app.py` — All backend routes & logic
- `SimplyAPI/templates/index.html` — Entire CRM dashboard (~6,000 lines Alpine.js SPA)
- `SimplyAPI/Funnels/` — Facebook lead scraping system
- `SimplyAPI/directives/` — SOPs for AI agents
- `SimplyAPI/execution/` — Deterministic Python scripts
- `Mrcar/` — AI pricing engine (Flask app)
- `camera app/web-deploy/` — Camera PWA (Vite build)

## Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per your directive. Only create new scripts if none exist. For Autodirecto specifically:
- Check `SimplyAPI/execution/` for backend scripts
- Check `camera app/execution/` for camera-related scripts
- Reuse existing API endpoints in `app.py` rather than creating new ones

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits/etc—in which case you check w user first)
- Update the directive with what you learned (API limits, timing, edge cases)
- Example: you hit an API rate limit → you then look into API → find a batch endpoint that would fix → rewrite script to accommodate → test → update directive.

**3. Update directives as you learn**
Directives are living documents. When you discover API constraints, better approaches, common errors, or timing expectations—update the directive. But don't create or overwrite directives without asking unless explicitly told to. Directives are your instruction set and must be preserved (and improved upon over time, not extemporaneously used and then discarded).

## Self-annealing loop

Errors are learning opportunities. When something breaks:
1. Fix it
2. Update the tool
3. Test tool, make sure it works
4. Update directive to include new flow
5. System is now stronger

## File Organization

**Deliverables vs Intermediates:**
- **Deliverables**: Google Sheets, Google Slides, or other cloud-based outputs that the user can access
- **Intermediates**: Temporary files needed during processing

**Directory structure:**
- `.tmp/` - All intermediate files (dossiers, scraped data, temp exports). Never commit, always regenerated.
- `execution/` - Python scripts (the deterministic tools)
- `directives/` - SOPs in Markdown (the instruction set)
- `.env` - Environment variables and API keys
- `credentials.json`, `token.json` - Google OAuth credentials (required files, in `.gitignore`)

**Key principle:** Local files are only for processing. Deliverables live in cloud services (Google Sheets, Slides, etc.) where the user can access them. Everything in `.tmp/` can be deleted and regenerated.

## Common Autodirecto Workflows

**Adding a New API Endpoint:**
1. Read existing patterns in `SimplyAPI/app.py`
2. Add route with proper error handling
3. Update PROJECT_OVERVIEW.md API routes table
4. If frontend needs it, add proxy in `src/app/api/`
5. Test with CRM dashboard or public site

**Modifying CRM Dashboard:**
1. Edit `SimplyAPI/templates/index.html` (Alpine.js + Tailwind)
2. No build step needed—just refresh browser
3. Use Alpine.js x-data, x-show, x-if for reactivity
4. API calls use fetch() with proper error handling

**Adding Fields to Vehicle Data:**
1. Update Supabase table schema (via SQL migration)
2. Add column to relevant tables: `consignaciones`, `listings`, `appraisals`
3. Update `app.py` endpoints that read/write this data
4. Update CRM forms in `index.html`
5. Update frontend components if public-facing
6. Update ChileAutos payload builder if field should publish

**Working with Photos:**
1. Camera PWA uploads to Supabase Storage: `vehicle-photos/{consignacion_id}/{filename}`
2. CRM reads from Supabase Storage via `GET /api/consignaciones/<id>/photos`
3. Catalog uses `photo_urls` array in listings table
4. Always use proper bucket paths and signed URLs

**ChileAutos Publishing:**
1. Vehicle must have: photos, complete specs, price, body_type, doors
2. Click "Publicar en ChileAutos" in CRM → calls `/api/consignaciones/<id>/publicar-chileautos`
3. Backend: OAuth2 token → build payload → PUT to ChileAutos API
4. Saves `chileautos_id` to consignaciones + listings tables
5. Auto-unpublishes on status change to vendida/cancelada

**Lead → Sale Flow:**
1. Lead created (Funnels, website, manual)
2. "Tasar con IA" → saves price to `crm_leads.ai_consignacion_price`
3. Wizard matches lead → creates consignación with saved price
4. Inspector photos → inspection report → contract PDF
5. Publish to web catalog → publish to ChileAutos
6. Buyer match → credit sim → purchase order → DTE

## Development Environment

**Running Locally:**
```bash
# Terminal 1: SimplyAPI backend
cd SimplyAPI
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python app.py  # → http://localhost:8080

# Terminal 2: Next.js frontend
npm install
npm run dev    # → http://localhost:3000
```

**Environment Variables:**
- `.env.local` (Next.js): Supabase keys, SimplyAPI URL
- `SimplyAPI/.env`: Supabase, Resend, Apify, MrCar, OpenAI keys
- ChileAutos creds: Stored in `crm_settings` table (NOT env vars)

**Testing:**
- CRM dashboard: http://localhost:8080
- Public site: http://localhost:3000
- Catalog: http://localhost:3000/catalogo
- Detail page: http://localhost:3000/catalogo/[id]

## Summary

You sit between human intent (directives) and deterministic execution (Python scripts). For Autodirecto specifically:

- **Read PROJECT_OVERVIEW.md first** — it's your complete knowledge base
- **Understand the business flow** — leads → valuation → consignment → photos → contract → publish → sale
- **Respect the architecture** — Next.js proxies, SimplyAPI brain, Supabase persistence
- **Follow business rules** — cached pricing, bidirectional sync, auto-unpublish logic
- Read instructions, make decisions, call tools, handle errors, continuously improve the system.

Be pragmatic. Be reliable. Self-anneal. Build features that help Chileans sell their cars efficiently.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dryanez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
