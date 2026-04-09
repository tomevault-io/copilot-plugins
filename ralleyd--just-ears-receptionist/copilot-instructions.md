## just-ears-receptionist

> AI voice receptionist system for Just Ears Clinic, handling inbound calls via Twilio + OpenAI Realtime API, with N8N workflow integration for appointment booking and clinic information.

# Just Ears Receptionist

AI voice receptionist system for Just Ears Clinic, handling inbound calls via Twilio + OpenAI Realtime API, with N8N workflow integration for appointment booking and clinic information.

## Project Structure

- `/server` — Node.js backend (Express, WebSocket, Twilio, OpenAI Realtime API)
- `/client` — React frontend (Vite, TanStack Query)
- `/shared` — Shared TypeScript types and Drizzle ORM schema
- `/n8n` — N8N workflow definitions (voice assistant functions, WhatsApp, weekly call reports)
- `/worker` — Cloudflare Worker (keep-alive)
- `/migrations` — Drizzle ORM database migrations
- `/dashboard` — Python subproject: Streamlit + Plotly client-facing call analytics dashboard

## Node.js Backend

- **Runtime:** Node.js 20, TypeScript 5.6
- **Server:** Express 4.21 on port 5000, WebSocket via `ws`
- **Voice AI:** OpenAI Realtime API (gpt-realtime-2025-08-28), G.711 u-law audio at 8kHz
- **Telephony:** Twilio SDK 5.10 — incoming calls, media streams, call transfers
- **Database:** PostgreSQL via Neon Serverless, Drizzle ORM
- **Auth:** Passport.js
- **Key env vars:** `OPENAI_API_KEY`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_NUMBER`, `N8N_WEBHOOK_BASE_URL`, `TRANSFER_NUMBER`, `DATABASE_URL`

### Call Flow
```
Incoming Call -> Twilio -> /api/incoming-call -> TwiML WebSocket URL
                              |
                         WebSocket /media-stream
                              |
                     OpenAI Realtime API <-> Server
                              |
                     N8N Webhooks (function calls: check_availability, create_appointment, get_info, transfer)
```

### Key Server Files
- `server/index.ts` — Express app init
- `server/routes.ts` — HTTP routes + WebSocket setup
- `server/realtime-client.ts` — OpenAI Realtime API integration
- `server/twilio-handler.ts` — TwiML generation, call transfers
- `server/functions.ts` — N8N webhook calls
- `server/config.ts` — Environment configuration

## Python Dashboard (`/dashboard`)

- **Framework:** Streamlit (current), planned migration path to Dash
- **Charts:** Plotly
- **Data source:** Twilio REST API via Python SDK (same data the N8N weekly report uses)
- **Theme:** Dark with soft blue (#7B8CDE), magenta (#C77DBA), yellow (#E8C547)
- **Python:** >=3.11, managed via pyenv virtualenv
- **Dependencies:** see `dashboard/pyproject.toml`
- **Implementation plan:** see `dashboard/implementation_plan.md`

### Dashboard Architecture
The `dashboard/data/` and `dashboard/charts/` packages have zero Streamlit imports — they are framework-agnostic. Only `app.py` imports Streamlit. This separation enables future migration to Dash without rewriting data or chart logic.

### Key Dashboard Files
- `dashboard/app.py` — Streamlit UI layer (only file that imports streamlit)
- `dashboard/dashboard/data/twilio_client.py` — Twilio data fetching (framework-agnostic)
- `dashboard/dashboard/data/metrics.py` — Metric computations (framework-agnostic)
- `dashboard/dashboard/charts/volume.py` — Plotly chart builders (framework-agnostic)
- `dashboard/dashboard/data/dummy.py` — Dummy data for development (remove after wiring real data)

### Running the Dashboard
```bash
cd dashboard
streamlit run app.py
```

## Conventions

- Node.js backend uses TypeScript throughout
- Dashboard Python code keeps data/chart logic framework-agnostic (no streamlit imports outside app.py)
- Twilio field names used consistently across dummy data and real data: `call_sid`, `start_time`, `price`, `status`, `parent_call_sid`
- Call transfers identified by non-empty `parent_call_sid` (child leg of a transferred call)
- Twilio `price` field: negative string or None — convert to positive float for display

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RalleyD)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RalleyD)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
