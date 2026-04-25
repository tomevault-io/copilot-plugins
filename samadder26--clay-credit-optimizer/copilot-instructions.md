## clay-credit-optimizer

> Full-stack dashboard that analyzes Clay enrichment credit usage and generates

# Clay Credit Optimizer

Full-stack dashboard that analyzes Clay enrichment credit usage and generates
AI-powered optimization recommendations.

## Tech Stack
- Backend: Python 3.11, Flask, Flask-CORS, Supabase Python client
- AI: LangGraph (3-node StateGraph), Anthropic Claude API (claude-sonnet-4-20250514)
- Frontend: React 19 + TypeScript, Vite, Tailwind CSS v4, shadcn/ui, Recharts, @dnd-kit
- Database: Supabase (Postgres)
- Deployment: Vercel (frontend), Render (backend), GitHub Actions (scheduled tasks)

## Commands
- Backend: `cd api && python app.py` (dev server on port 5000)
- Frontend: `cd frontend && npm run dev` (Vite dev server on port 5173)
- Install backend deps: `cd api && pip install -r requirements.txt`
- Install frontend deps: `cd frontend && npm install`

## Architecture
- `/api` — Flask backend
- `/frontend` — React frontend (Vite + shadcn/ui)
- `/.github/workflows` — GitHub Actions

## Important
- NEVER commit .env files
- Use http://localhost:5173 for CORS origin in development
- Supabase RLS policies must allow anon key access for development
- Use claude-sonnet-4-20250514 as the model string for Claude API calls

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samadder26) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
