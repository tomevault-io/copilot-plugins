## alphasynth-equity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start Express + Vite dev server together (tsx server.ts)
npm run build    # Vite production bundle → dist/
npm run start    # Production mode: NODE_ENV=production node server.ts
npm run lint     # TypeScript type-check only (tsc --noEmit, no test suite exists)
```

## Architecture

This is a single-process full-stack app: `server.ts` runs an Express API and embeds the Vite dev server as middleware (or serves `dist/` in production). There is no separate frontend dev server — everything runs on port 3005.

### Backend — `server.ts`

All AI, scraping, and email logic lives here. Clients are lazily initialised once via getter functions (`getVertexAI`, `getResend`, `getFirecrawl`) and cached in module-level variables.

**API routes:**
| Route | Purpose |
|---|---|
| `GET /api/health` | Liveness + config check |
| `GET /api/market-intel` | Trending NSE stocks via Gemini + Google Search grounding |
| `POST /api/pipeline/scrape` | Firecrawl scrape/search for a ticker URL |
| `POST /api/pipeline/price` | Live NSE price via Gemini search grounding |
| `POST /api/pipeline/analyze` | Two-stage AI analysis (research / filings / move / earnings modes) |
| `POST /api/pipeline/peers` | Sector peer benchmarking data |
| `POST /api/portfolio/analyze` | Portfolio audit via Gemini |
| `POST /api/email/briefing` | Send report email via Resend |

**Two-stage analysis pattern** (`/api/pipeline/analyze`): Stage 1 calls `gemini-2.5-flash` with `googleSearch` grounding to get raw narrative. Stage 2 calls the model again with a strict JSON response schema (`responseMimeType: "application/json"`) to extract `report` (Markdown string) and `benchmarking` (numeric array). All routes have hardcoded fallback payloads for when the AI call fails.

**`sanitizeGroundingJson`**: Strips markdown fences, citation markers like `[1]`, `₹`, and `%` from Vertex AI responses before `JSON.parse`. Apply this to every raw text response before parsing.

### Frontend — `src/App.tsx`

Single large React file (~300KB). Contains all components, state, Firebase logic, and UI inline. Major sections:
- Firebase Auth (Google sign-in via popup) and Firestore reads/writes
- `MarkdownComponents` map for custom-styled `react-markdown` rendering
- Tabs: `news`, `equity`, `filings`, `marketing`, `community`, `portfolio`
- Zerodha broker integration state (API key stored in `localStorage`)
- `isMarketOpen()` — checks IST timezone for NSE hours (Mon–Fri 9:15–15:30)
- Recharts (`PieChart`, `BarChart`) for peer benchmarking visualisation

### Environment Variables (`.env`)

```
GCP_PROJECT_ID=          # Google Cloud project with Vertex AI enabled
GCP_REGION=us-central1
GOOGLE_APPLICATION_CREDENTIALS=gcp-key.json   # Path to service account JSON
FIRECRAWL_API_KEY=       # Firecrawl v4 API key
RESEND_API_KEY=          # Resend email API key
PORT=3005
NODE_ENV=development
```

`gcp-key.json` must be present at the project root (or the path set in `GOOGLE_APPLICATION_CREDENTIALS`). The `.env` values for `GCP_PROJECT_ID` and `GOOGLE_APPLICATION_CREDENTIALS` have leading spaces — trim them if editing manually.

### Firebase

Config is in `firebase-applet-config.json` (committed, not secret — it's a client-side Firebase config). Firestore uses a non-default database ID (`firestoreDatabaseId` field). Security rules are in `firestore.rules` and follow a strict deny-all default with explicit per-collection rules for `users`, `reports`, `subscribers`, and `user_portfolio`.

### Key Dependencies

- `@google/genai` v2 — unified Google Gen AI SDK; use `new GoogleGenAI({ vertexai: true, project, location })` for Vertex AI. All AI calls go through `ai.models.generateContent({ model, contents, config })`. Response text is accessed via `response.text`.
- `@mendable/firecrawl-js` v4 — scrape API has changed across major versions; use `scraper.scrape(url, { formats: ['markdown'] })` and `scraper.search(query, { limit })`. Search returns `{ web: Document[], news: Document[] }` — NOT `.data` (accessing `.data` throws).
- `tsx` — runs `server.ts` directly in dev without a compile step
- `node-domexception` — local package override in `local-packages/` to fix a compatibility issue

---
> Source: [srscoll-lab/alphasynth-equity](https://github.com/srscoll-lab/alphasynth-equity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
