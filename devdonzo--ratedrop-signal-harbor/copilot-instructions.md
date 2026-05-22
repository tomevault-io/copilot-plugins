## ratedrop-signal-harbor

> This repository is a standalone product called RateDrop.

# AGENTS

This repository is a standalone product called RateDrop.

Do not frame it as related to ASSIST, prior demos, or internal prototypes.
Do not add ASSIST references to UI copy, docs, comments, or commit messages.

## Product Summary

RateDrop is a hackathon MVP that:

1. Accepts a telecom bill PDF or image
2. Extracts structured bill facts
3. Builds a deterministic negotiation plan
4. Starts a sandbox Twilio voice flow
5. Streams transcript updates in the UI
6. Persists bill, negotiation, and transcript state in MongoDB
7. Shows a deterministic savings result page

## Current Architecture

- Frontend: Next.js 15 in the repo root
- Backend: FastAPI in `backend/app/`
- Database: MongoDB Atlas
- Model: Gemini 2.5 Flash
- Voice: Twilio Programmable Voice

## Repo Layout

- `app/`: Next.js routes
- `components/`: frontend UI components
- `lib/`: frontend API client and types
- `backend/app/main.py`: FastAPI routes
- `backend/app/services/extraction.py`: Gemini extraction and demo fixtures
- `backend/app/services/negotiation.py`: deterministic negotiation engine
- `backend/app/services/twilio_voice.py`: Twilio outbound call and callbacks
- `backend/app/db/mongo.py`: Mongo client and indexes
- `scripts/smoke_mvp.py`: happy-path smoke test
- `scripts/backend_sanity.py`: backend validation and edge-case check

## Non-Negotiable Product Constraints

- Keep the product consumer-facing and standalone
- Do not rely on pure LLM improvisation for negotiation logic
- Use Gemini for extraction and phrasing only
- Keep final math deterministic and code-driven
- Prefer a reliable demo path over extra features
- Do not break the existing API contracts unless the whole stack is updated together

## Backend Ownership

Backend owns:

- request validation
- bill extraction
- scenario selection
- negotiation turn plan
- deterministic savings math
- transcript persistence
- SSE event publishing
- Twilio call start and callback handling
- MongoDB document shapes

Frontend should not reimplement or override backend logic.

## Frontend Ownership

Frontend owns:

- page layout
- visual hierarchy
- styling
- interaction polish
- loading, empty, and error states
- mobile responsiveness
- rendering backend data clearly

Frontend should consume the existing API and type contracts rather than inventing new ones.

## API Contracts To Preserve

- `POST /api/bills/upload`
- `GET /api/demo-bills`
- `POST /api/bills/demo/{scenarioId}`
- `GET /api/bills/{id}`
- `GET /api/negotiations?limit=N`
- `POST /api/negotiations`
- `POST /api/negotiations/{id}/start`
- `GET /api/negotiations/{id}`
- `GET /api/negotiations/{id}/transcript`
- `GET /api/negotiations/{id}/events`
- `POST /twilio/voice/negotiations/{id}`
- `POST /twilio/status/{id}`

Primary frontend contract files:

- `lib/api.ts`
- `lib/types.ts`

If these change, update the frontend and backend together.

## Local Run Commands

Install:

```bash
npm install
python3 -m pip install -r backend/requirements.txt
```

Run backend:

```bash
npm run dev:api
```

Run frontend:

```bash
npm run dev
```

Open:

```text
http://127.0.0.1:3000
```

## Environment Variables

Required:

- `MONGODB_URI`
- `GEMINI_API_KEY`
- `GEMINI_MODEL`
- `TWILIO_ACCOUNT_SID`
- `TWILIO_AUTH_TOKEN`
- `TWILIO_PHONE_NUMBER`

Optional but important:

- `TWILIO_VERIFY_ORIG_NUMBERS`
- `TWILIO_SANDBOX_TO_NUMBER`
- `PUBLIC_BASE_URL`
- `FRONTEND_BASE_URL`
- `NEXT_PUBLIC_API_BASE_URL`

Behavior notes:

- If `PUBLIC_BASE_URL` is missing, the backend falls back to simulated call mode
- Twilio trial mode requires verified destination numbers

## Reliability Notes

The code is in good hackathon-demo shape, but there are still operational dependencies:

- local frontend process must stay running
- local backend process must stay running
- public callback tunnel must stay alive for real Twilio callbacks
- verified Twilio destination must be reachable

Do not describe the system as production-grade.

## Current Verified Behavior

Verified in this repo:

- frontend builds cleanly
- backend compiles cleanly
- invalid IDs return `400`
- malformed negotiation create payload returns `400`
- duplicate start requests do not duplicate transcript runs
- in-progress negotiations resume after backend restart
- smoke path completes end-to-end
- backend sanity script passes

## Safe Change Boundaries

Safe changes:

- CSS and layout
- page composition
- component refactors that preserve behavior
- copy improvements that keep RateDrop branding
- better client-side loading and error handling

Risky changes:

- changing `lib/types.ts`
- changing `lib/api.ts` request/response assumptions
- changing Mongo document shapes
- changing negotiation math
- changing SSE payloads
- changing Twilio webhook paths

## Required Validation After Meaningful Changes

For backend changes:

```bash
python3 -m py_compile backend/app/main.py backend/app/services/negotiation.py backend/app/services/twilio_voice.py backend/app/db/mongo.py backend/app/models/schemas.py
python3 scripts/backend_sanity.py
```

For frontend changes:

```bash
npm run build
```

For end-to-end checks:

```bash
python3 scripts/smoke_mvp.py
```

## If You Are Acting As An Agent

- Read `plan.md` first for product intent
- Keep the product branded as RateDrop
- Favor small, production-minded changes
- Use the simplest path that keeps the demo reliable
- Do not remove unrelated user changes
- Do not claim functionality is flawless without verification

---
> Source: [DevDonzo/ratedrop-signal-harbor](https://github.com/DevDonzo/ratedrop-signal-harbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
