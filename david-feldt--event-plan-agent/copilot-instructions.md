## event-plan-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Woodsy** — AI agent that coordinates social plans with friends over voice and WhatsApp/SMS. Calls users weekly, learns availability, proposes plans, sends invites. React Native app shows plans and pending invites.

## Commands

```bash
# Development
./dev.py server          # Start FastAPI backend (port 8000, auto-reload)
./dev.py tunnel          # Start ngrok tunnel for Twilio webhooks
./dev.py dev             # Server + ngrok together (typical dev setup)
./dev.py mobile          # Start Expo app (from mobile/)
./dev.py seed            # Seed database via migrations/seed.py

# Simulator (test without external services)
./dev.py simulate                              # Engine/scorer offline only
./dev.py simulate --llm                        # + Gemini conversation
./dev.py simulate --text --test               # In-memory text chat (no Supabase)
./dev.py simulate --text --test --phone +16471234567  # With area code inference
./dev.py simulate --text                       # Interactive WhatsApp-like chat (needs Supabase)
./dev.py simulate --demo                       # Simple 2-user demo (Alice ↔ Bob) — auto-plays a message grid showing engine matching; persists to .woodsy_demo.db (real agent + queue, needs GOOGLE_API_KEY; runs with WOODSY_DEMO=1 to skip memory-extraction + web-search calls)
./dev.py simulate --scenarios                  # Same scripted scenarios, silent + engine timeline (persists to .woodsy_demo.db, needs GOOGLE_API_KEY)
./dev.py simulate --multi                      # Interactive multi-user chat: type "1 <msg>" to speak as user 1

# Trigger outbound call manually
curl -X POST "http://localhost:8000/api/voice/calls/outbound?user_id=<uuid>"
```

There is no traditional test suite — `tests/` is empty. Use simulator modes for isolated testing.

## Architecture

### Channels

Two independent ingress paths share the same agent and tools:

| Channel | Entry Point | Transport |
|---------|-------------|-----------|
| Voice | `GET /api/voice/twiml` → `WS /api/voice/stream` | Twilio Media Streams + Deepgram STT/TTS |
| WhatsApp/SMS | `POST /api/messaging/inbound` | Twilio webhook → TwiML reply |

An **admin dashboard** at `GET /admin` (`app/api/admin.py`, Jinja2 templates in `templates/`) renders a read-only view of the engine queue, users + intent/availability, active suggestions, and plans. All queries are wrapped in `_safe_query` so missing tables degrade gracefully.

### Voice Pipeline (end-to-end)

Scheduler or manual trigger → Twilio calls user → Twilio fetches TwiML → Twilio opens WebSocket to `/api/voice/stream` → audio frames streamed to Deepgram STT → transcripts fed to `VoicePipeline` (Gemini-2.5-Flash + tool loop) → response text → Deepgram TTS → mulaw audio chunks → back to Twilio → phone. On `stop` event: full transcript saved, memories extracted async.

### Text Pipeline (end-to-end)

Twilio webhook POST → `route_message()` in `app/agent/router.py` checks user state:
1. Unknown phone → start onboarding (`app/agent/onboarding.py`, multi-step Gemini conversation)
2. Onboarding in progress → continue onboarding
3. Pending RSVP → detect intent (`app/agent/rsvp.py`) → update `plan_members.invite_status`
4. Default → Woodsy agent (`app/agent/woodsy.py`, Gemini + tool loop)

Response wrapped in TwiML and returned synchronously to Twilio.

### Agent Tools (`app/agent/tools.py`)

7 tools available in both voice and text agents:

| Tool | Purpose |
|------|---------|
| `get_user_context` | User profile, memories, recent plans |
| `record_availability` | Save availability windows (writes to `users.availability_windows`, sets `plan_intent`, pushes an `availability_updated` engine job) |
| `search_events` | Match events to interests via pgvector |
| `search_web` | Brave Search for local events/activity ideas (needs `BRAVE_API_KEY`) |
| `propose_plans` | Get new-plan clusters and join-existing suggestions |
| `send_invite` | Invite friends (prioritized: friends first) |
| `update_memory` | Store facts (preference/constraint/feedback/relationship_note) |

Tool calling loop: max 5 iterations, results fed back to Gemini as context.

### Planning Engine (`app/engine/`)

- `scorer.py` — Compatibility scoring: interest similarity (pgvector cosine), availability overlap, location, friendship strength
- `planner.py` — Identifies clusters of 3+ compatible users for new plans; finds open plans to join
- `availability.py` — Fuzzy time parsing ("Saturday afternoon" → ISO windows) + overlap detection
- `lifecycle.py` — Plan state machine: `FORMING → PROPOSED → CONFIRMED → LIVE → PAST/CANCELLED`
- `invites.py` — Prioritized invite sequencing
- `queue.py` — Supabase-backed job queue (`engine_jobs` table) that decouples engine runs from the request/scheduler.

**Engine is triggered by intent, not by a timer.** When a user expresses plan intent (text agent's keyword `_detect_and_queue_intent` in `woodsy.py`, or `record_availability`), a job is pushed via `push_job(user_id, trigger)` (dedup'd against pending jobs for that user). A long-running `run_queue_worker()` task (started in `main.py` lifespan, polls every 30s) pulls pending jobs and recomputes new-plan + join suggestions for the user. This replaces the old periodic `run_suggestion_generation` cron.

### Database (`app/db/`)

Supabase (Postgres + pgvector). Two clients:
- `get_anon_client()` — RLS enforced (user-facing)
- `get_service_client()` — RLS bypassed (agent tools always use this)

`mock_supabase.py` provides in-memory drop-in for simulator `--test` mode, toggled by `use_mock_db()`.

Key tables: `users`, `conversations`, `agent_memory` (pgvector), `user_preferences` (pgvector), `plans`, `plan_members`, `plan_suggestions`, `friendships`, `engine_jobs` (queue). `users` also carries `plan_intent`/`plan_intent_at`, `availability_windows` (jsonb), and `vibe` (see migration 005).

Custom RPC: `match_memories()` and `match_users()` for pgvector semantic search.

### Scheduler (`app/agent/scheduler.py`, via APScheduler in `main.py`)

User-facing outreach only — the planning engine is NOT scheduled here (see queue above).

- Thursday 6 PM: `run_weekly_calls()` — outbound calls to all users
- 3 AM daily: `run_plan_expiry()` — move past-date plans to "past"
- 10 AM daily: `run_engagement_nudges()` — WhatsApp nudge to users with stale (>3d) plan intent or no chat in 5+ days

## Key Notes

**Embeddings are currently disabled** — `embed_text()` in `app/agent/memory.py` returns `None` because `text-embedding-004` was deprecated. Semantic memory recall and pgvector search skip gracefully. Migration needed to `text-embedding-005` or `gemini-embedding-exp`.

**Dynamic system prompts** — Built per-call/conversation with user context, memories, recent plans, friend activity, and plan suggestions. See `app/agent/conversation.py`.

**Onboarding completion detection** — Gemini response checked for phrases like "all set", "you're set" to detect when onboarding is done (fragile — watch for regressions when changing onboarding prompts).

**Voice testing locally** requires ngrok + Twilio phone number configured with your ngrok URL as the TwiML webhook. WhatsApp testing requires Twilio WhatsApp sandbox similarly configured.

**Simulation logger** (`app/sim_logger.py`) — a global, opt-in singleton for `--scenarios` mode. Tool handlers emit structured engine events (SCORE/CLUSTER/SUGGEST/INVITE/SPARK/EXPAND/INTEREST) via `get_sim_logger()`, which returns `None` (zero-cost) unless `activate_sim_logger()` was called. Used to render an engine timeline after a scenario run.

## Environment Variables

```
SUPABASE_URL, SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY
GOOGLE_API_KEY            # Gemini-2.5-Flash
TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_PHONE_NUMBER
TWILIO_WHATSAPP_NUMBER    # e.g. whatsapp:+14155238886
DEEPGRAM_API_KEY          # STT + TTS (voice only)
BRAVE_API_KEY             # search_web tool (optional)
SERVER_URL                # Public URL for Twilio webhooks (default: http://localhost:8000)
```

## Database Migrations

Run in order in Supabase SQL Editor:
1. `migrations/001_initial_schema.sql`
2. `migrations/002_remove_vapi.sql`
3. (Optional) `migrations/003_interest_matching.sql`, `migrations/004_user_signals.sql`
4. `migrations/005_queue_and_profile.sql` — `engine_jobs` queue table + `users` profile columns (`plan_intent`, `availability_windows`, `vibe`)

Then seed: `./dev.py seed`

---
> Source: [David-Feldt/event-plan-agent](https://github.com/David-Feldt/event-plan-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
