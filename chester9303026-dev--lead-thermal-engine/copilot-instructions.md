## lead-thermal-engine

> Context for the Claude Code agent working in this project.

# CLAUDE.md — Lead Thermal Engine

Context for the Claude Code agent working in this project.

## What this project is
A two-part lead-generation tool for a design & advertising studio that sells
websites, Instagram management, and ads to wellness / massage / spa businesses.

1. **collector.py** — pulls businesses from the Google Places API (New) and
   writes `leads.csv` (name, city, has_website, rating, reviews, phone, a
   chain-size heuristic). Instagram / ad / visual fields are left blank for
   manual enrichment on the short list of top leads.
2. **React app (`src/App.jsx`)** — imports `leads.csv`, scores each business by
   *Need* (how much they need a studio) × *Fit* (can they pay), ranks them, and
   writes a targeted outreach opener per lead. Message generation goes through
   **`server.js`**, a local proxy that holds the Anthropic key server-side.

Pipeline: `collector.py` → `leads.csv` → import into the app → ranked leads + openers.

## Files — do NOT rewrite these, they are already wired
- `src/App.jsx` — the engine. Calls `/api/generate` for message generation.
- `server.js` — Anthropic proxy. Reads `ANTHROPIC_API_KEY` from `.env`.
- `vite.config.js` — proxies `/api` → `http://localhost:8787`.
- `collector.py` — reads `GOOGLE_MAPS_API_KEY` from `.env`.

## Setup steps (do these in order)
1. Create `.env` by copying `.env.example`. Then ASK the user for their
   `GOOGLE_MAPS_API_KEY` (Google Cloud → enable "Places API (New)") and
   `ANTHROPIC_API_KEY` (console.anthropic.com). Do not invent or commit keys.
2. Install Node deps: `npm install`. If a pinned version fails, bump it to the
   latest compatible release and retry — do not change the project structure.
3. Set up Python for the collector:
   `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
   (On Windows: `.venv\Scripts\activate`.)
4. Start the app: `npm run dev`. This runs Vite (http://localhost:5173) and the
   proxy (http://localhost:8787) together. Open the localhost URL.
5. To gather leads: edit `KEYWORDS` and `CITIES` at the top of `collector.py`
   for the user's target area, then run `python collector.py` (venv active).
   Import the resulting `leads.csv` in the running app.

## Notes
- Never put the Anthropic key in frontend code — it stays in `.env` / `server.js`.
- The app works without an Anthropic key too: scoring and ranking are fully
  local; only the AI opener needs the proxy (there's a templated fallback if the
  call fails).
- Keep `.env` and `leads.csv` out of git (already in `.gitignore`).

---
> Source: [chester9303026-dev/lead-thermal-engine](https://github.com/chester9303026-dev/lead-thermal-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
