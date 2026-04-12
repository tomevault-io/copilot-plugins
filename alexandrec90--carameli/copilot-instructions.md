## carameli

> A self-hosted VoIP microservice. Manages phone lines, extensions, SMS, call recording, and call tracking via a REST API.

# Carameli

A self-hosted VoIP microservice. Manages phone lines, extensions, SMS, call recording, and call tracking via a REST API.

## Tech Stack

| Layer | Choice |
| --- | --- |
| Language | Python 3.12 |
| Framework | FastAPI |
| Background jobs | ARQ (async, Redis-backed, separate worker process) |
| Database | PostgreSQL 18 |
| ORM / Migrations | SQLAlchemy 2 (async) + Alembic |
| Call engine | Jambonz (self-hosted, on FreeSWITCH) |
| Carrier / SIP trunk | Telnyx (wholesale) — provider-abstracted |
| Media storage | S3-compatible blob (local disk in dev) |
| Container | Docker + Docker Compose |
| Auth | Bearer API key (`Authorization: Bearer <key>`) |
| Tests | pytest + pytest-asyncio |

## Local Development

```bash
# Start everything (includes Jambonz + FreeSWITCH + rtpengine)
docker compose up

# Apply DB migrations
docker compose exec app alembic upgrade head

# Run tests
docker compose exec app pytest

# Expose webhook endpoints publicly (Jambonz + Telnyx need to reach Carameli)
ngrok http 8000
# Then set JAMBONZ_WEBHOOK_BASE_URL and TELNYX_WEBHOOK_BASE_URL in .env to the ngrok HTTPS URL
```

## Environment Variables

See `.env.example` for all vars. All settings are loaded via pydantic-settings in `app/core/config.py`.

## Call Tracking

The active call engine (Jambonz) fires a status webhook when a call ends. Carameli:

1. Validates the webhook signature
2. Writes the raw event to the `call_events` PostgreSQL table
3. Matches it to a call record and updates talk time / call attempt counters

APScheduler runs a retry job every 30 seconds for any failed writes.

## VanillaLand Reference

`../VanillaLand/` is the legacy .NET/SQL Server CRM+VoIP monolith that Carameli is designed to
replace at the telephony layer. Use it to understand existing feature contracts before implementing
or extending Carameli endpoints. Everything outside the table below is excluded from the context
window via `.claudeignore`.

### Technology mapping

| VanillaLand (legacy) | Carameli equivalent |
| --- | --- |
| ConnectMeVoice (CMV) / CloudLi | Jambonz call engine (`app/services/providers/engine/jambonz.py`) |
| Telnyx carrier (same) | Telnyx carrier provider (`app/services/providers/carrier/telnyx.py`) |
| `tblPhoneNumber` / `VoIPEntities` | `phone_lines` + `extensions` DB tables |
| `tblCallHistory` | `call_events` DB table |
| `SMSWS.asmx` web service | `/vsapi/1.0.0/VsMessaging/Sms/` routes |
| `CMVCallInfo.asmx` web service | `/webhooks/jambonz/call-status` webhook |
| `VoiceMailDropHistory` | voicemail_drop service + `/vsapi/1.0.0/VsMessageDrop` |
| IntellectiveRouting / CallerRouting | SCI routing (`app/api/vsapi/sci.py`) |
| DID provisioning (phone number lifecycle) | `app/services/did_manager.py` |

See `.claude/rules/vanillaland-paths.md` for the full path reference table.

## Front-End (Carameli UI)

The frontend uses a **skin system** that fully decouples data logic from visual layout.
Skins can use completely different tech stacks (CSS, Three.js, etc.) without touching shared code.
See `.claude/rules/skin-architecture.md` for the authoritative spec.

| Layer | Choice |
| --- | --- |
| Component framework | React (TSX) |
| Build / bundler | Vite (per-skin code splitting via dynamic import) |
| Active skin | `carameli` (3D canvas, React Three Fiber) |

### Frontend Layout

```text
frontend/src/
  hooks/               # Data layer — one hook per page, no JSX
    useDashboard.ts
    usePhoneLines.ts
    useExtensions.ts
  skins/
    types.ts           # Skin / SkinViews TypeScript interfaces
    registry.ts        # Dynamic import map (one Vite chunk per skin)
    context.tsx        # SkinProvider + useSkin()
    carameli/          # Active skin (3D "Liquid Candy Maximalism")
      index.ts         # Skin entry point / chunk boundary
      Layout.tsx
      views/           # Skin-specific page renderers (props only, no API calls)
  pages/               # Thin orchestrators: call hook → call useSkin() → render view
  api/                 # API client + TypeScript types
  lib/                 # logger, utilities
```

### Skin Design Constraints (carameli skin)

See `.claude/rules/skin-carameli.md` for the full 3D canvas spec. Quick reference:

- Entire UI inside `<Canvas>` (React Three Fiber) — no CSS-styled DOM for primary surfaces
- `MeshPhysicalMaterial` on all interactive surfaces — `meshBasicMaterial` forbidden
- Real panel depth via `RoundedBox` (z ≥ `0.18`) — no flat planes as UI panels
- 3D extruded text (`Text3D` with `bevelEnabled`, `height ≥ 0.2`)
- Spring physics for all motion (`@react-spring/three`) — no CSS transitions
- Warm lights only — minimum 3 colored point lights, no cold/white lights
- Fluid vertex-shader background, always moving
- Post-processing always on: Bloom + ChromaticAberration + Vignette

Use the `add-ui-component` skill when building new components to get a step-by-step
checklist and copy-paste examples for buttons, cards, modals, and nav elements.

## Logging

All backend and frontend activity is written to a single rotating log file.
See `.claude/rules/logging.md` for the full spec.

**Quick reference:**

- Log file: `logs/runtime/carameli.log` (10 MB cap, 5 backups)
- Format: `YYYY-MM-DD HH:MM:SS.mmm | LEVEL | module:line | message`
- Every Python module: `logger = logging.getLogger(__name__)` at module scope
- Every route handler: log entry at `INFO`, 404s at `WARNING`, errors at `ERROR`
- Frontend: `import { logger } from '../lib/logger'` — auto-ships to backend log file
- Never log secrets (`api_key`, credentials)
- A global `@app.exception_handler(Exception)` in `app/main.py` ensures all unhandled 500s are written to the log file with full stack traces — **do not remove it**; it is the primary signal for AI-assisted debugging via `logs/runtime/carameli.log`

## Tooling

See `.claude/rules/tooling.md` for VS Code task script conventions.

- Task scripts live in `scripts/` and must be PowerShell (`.ps1`) — not Bash/`.sh`
- Always invoke scripts with `pwsh` (PowerShell 7), never `powershell` (Windows PowerShell 5.1)
- Use only ASCII characters in `.ps1` files — no em-dashes, curly quotes, or other non-ASCII (they cause parse errors when the file encoding is misread)
- **Never run `docker` or `docker compose` commands directly** — provide the commands for the user to run instead

## Guardrails (Vibe-Code Safety)

This project is vibe-coded. The rules below exist to catch the mistakes that vibe coding produces most often. **Treat every item as mandatory** — not aspirational.

### Auth & customer scoping

- Every new route handler **must** depend on the auth dependency (`get_auth_context`) and call `enforce_customer_scope()` to filter queries by `vs_customer_id`
- Never return data without scoping it to the authenticated customer — this is a multi-tenant system
- If a route is intentionally public (e.g., health check), document why auth is skipped with a comment

### Alembic migrations

- Any change to a SQLAlchemy model (new column, changed type, new table, dropped column) **must** include a corresponding Alembic migration in the same commit
- Generate with: `alembic revision --autogenerate -m "short_description"` — then review the generated file before committing
- Every migration must have a working `downgrade()` — never leave it as `pass`
- See `.claude/rules/migrations.md` for naming and safety conventions

### Provider abstraction

- Services and route handlers import **only** from `app/services/providers/base.py` (the Protocol interfaces)
- Never import concrete providers (`telnyx.py`, `jambonz.py`) outside of `factory.py` and tests
- When adding a new provider method, add it to the Protocol first, then implement it in every concrete provider

### Pydantic schemas for all endpoints

- Every route handler must declare a Pydantic model for its request body and response (`response_model=`)
- Never return raw dicts or untyped data from an endpoint — this leaks internal fields and skips validation
- Schemas live in `app/schemas/` — one module per resource

### Async discipline

- The entire backend is async (FastAPI + async SQLAlchemy + httpx). **Never use blocking calls** in async handlers:
  - No `time.sleep()` — use `asyncio.sleep()`
  - No `requests.get()` — use `httpx.AsyncClient`
  - No synchronous file I/O in hot paths — use `aiofiles` or offload to a thread with `asyncio.to_thread()`
- If you must call a blocking library, wrap it in `asyncio.to_thread()`

### Webhook signature validation

- Every webhook endpoint (Jambonz, Telnyx, or any future provider) **must** validate the request signature before processing the payload
- Never trust webhook data without verification — see `.claude/rules/webhooks.md` for the pattern

### Dependency management

- When adding a new pip package, add it to `requirements.txt` (runtime) or `requirements-dev.txt` (dev/test only) in the same commit
- Use the `audit-deps` skill periodically to detect unused, missing, or misplaced dependencies
- Never leave an import that depends on a package not listed in requirements

## Testing Strategy

**Mandatory test coverage**: Every code change that adds or modifies functionality **must** include corresponding tests in the same commit. This is non-negotiable — the project is vibe-coded and tests are the primary safety net against regressions.

- When creating a new endpoint, service, or utility: write unit tests covering the happy path, error cases, and edge cases **before** considering the task complete
- When modifying existing code: update or add tests to cover the changed behavior; never assume existing tests are sufficient
- When fixing a bug: write a regression test that reproduces the bug first, then fix it
- Proactively use the `make-tests` skill to identify and fill coverage gaps after any significant change
- Aim for meaningful coverage — test business logic, validation, error handling, and provider interactions, not just that code runs without crashing

### Test types

- **Unit tests**: pytest with mocked provider interfaces (`unittest.mock.patch` at the `CarrierProvider` / `CallEngineProvider` boundary — never mock internal SDK details)
- **Integration tests**: Telnyx sandbox credentials + a local Jambonz instance (no real charges)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexandrec90)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexandrec90)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
