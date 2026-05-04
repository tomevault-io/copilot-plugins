## trifit

> TriFit: A web app that reads Garmin .FIT activity files, visualizes

## Project Overview
TriFit: A web app that reads Garmin .FIT activity files, visualizes
performance data, generates AI-powered triathlon training plans,
and adapts schedules to real life.

## Tech Stack
- Frontend: Static HTML + Tailwind CSS (CDN) — hosted on GitHub Pages
- Backend Proxy: Node.js + Express — hosted on Render.com
- Database: Supabase (PostgreSQL + Auth + RLS)
- UI: Tailwind CSS + shadcn/ui
- Charts: Chart.js (CDN)
- AI: to be discussed later
- File Parsing: fit-file-parser

## Architecture
- Frontend (GitHub Pages): `index.html`, `analysis.html`, `upload.html`, `setup.html`, `login.html`
- Backend (Render.com): `server/` directory — proxies third-party API calls, keeps secrets server-side
- Database & Auth (Supabase Cloud): PostgreSQL + RLS + Auth — frontend connects directly via anon key
- Shared module: `auth.js` — authentication, session management, database CRUD, nav injection
- See docs/supabase-setup.md for database schema and setup guide

## Conventions
- All components in src/components/, grouped by feature
- All API routes in src/app/api/
- Database queries go through src/lib/db/ — never call Supabase directly from components
- Environment variables in `server/.env.local`, never committed
- `.env.example` files are committed (variable names only, no values)
- `.gitignore` must always include `.env.local`, `.env`, `.env.*.local`
- Commit messages: conventional commits (feat:, fix:, docs:, refactor:)
- **API Key Security**: API keys (e.g. OpenWeatherMap) are NEVER stored in frontend code, localStorage, or any client-accessible location. All third-party API keys are stored as environment variables on the backend server (Render.com). The frontend calls the backend proxy (`/api/weather`), which injects the API key server-side before forwarding requests to upstream APIs. The backend enforces CORS whitelisting, rate limiting, input validation, and security headers (Helmet). No API key is ever transmitted to or visible in the browser.
- **Weather-Aware Training**: Weather data from OpenWeatherMap API is displayed on the Dashboard (fetched via backend proxy). When rain is detected, the training plan logic checks the user's equipment profile (indoor trainer / treadmill). If the user has an indoor bike trainer, cycling can proceed indoors on rainy days; if they have a treadmill, running can proceed indoors. Otherwise, outdoor bike and run sessions are flagged with a rain warning suggesting an indoor alternative or rest day.
- **XSS Prevention**: Never use `innerHTML` with user-generated content. Use `textContent` or DOM API (`createElement` / `appendChild`) to inject user input into the page.
- **Supabase Auth & RLS**: Supabase anon key is public and safe to include in frontend code. All data security is enforced by Row Level Security (RLS) policies — every table has RLS enabled, users can only read/write their own data. The `settings_history` table is append-only (no UPDATE/DELETE in RLS) to serve as an audit log.
- **Anti-FOUC**: All protected pages have `<main style="display:none">`. The `auth.js` module reveals content only after confirming the user is authenticated, preventing unauthenticated content flash.
- **Login Security**: Frontend enforces password strength (8+ chars, letters + numbers), email confirmation via Supabase, and a brute-force lockout (5 failed attempts → 60s cooldown).

## Current Status
- [x] M0: Environment setup 
- [x] M1: Landing Page + Prompt skills 
- [x] M2: API integration (weather)
- [x] M3: Backend + Auth + Database
- [ ] M4: Data visualization (Dashboard) ← YOU ARE HERE
- [ ] M5: .FIT file parsing
- [ ] M6: AI training plan + Coach chat
- [ ] M7: Polish + Deploy + Monitor
- [ ] M8: Retrospective + Playbook

---
> Source: [jikonka/trifit](https://github.com/jikonka/trifit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
