## kraken-chatbot

> This file provides guidance to Claude Code when working with the KRAKEN Chatbot codebase.

# CLAUDE.md

This file provides guidance to Claude Code when working with the KRAKEN Chatbot codebase.

## Commands

```bash
# Frontend (React)
npm run dev          # Start dev server (Vite HMR on port 5173)
npm run build        # Build client (Vite) → dist/
npm run check        # TypeScript type checking

# Backend (FastAPI/Python)
cd backend
uv sync              # Install/update Python dependencies
uv run uvicorn src.kestrel_backend.main:app --reload --host 127.0.0.1 --port 8000

# Testing
# Use `python -m pytest` (not bare `uv run pytest`): the venv path contains
# spaces, which breaks the console-script shebang and silently falls back to
# system Python → misleading ModuleNotFoundError. See
# docs/solutions/developer-experience/pytest-venv-path-spaces-module-invocation-2026-06-01.md
cd backend && uv run python -m pytest tests/ -v -m "not integration"
```

## LangGraph Studio (Local Development)

Visual debugger for the KRAKEN discovery pipeline.

### Setup

1. Ensure `backend/.env` exists: `cp backend/.env.example backend/.env` and fill in API keys
2. `cd backend && uv run langgraph dev`
3. Open the Studio URL printed in the terminal (requires a free [LangSmith](https://smith.langchain.com) account)
4. Graph topology renders automatically; invoke with a test query to see node execution

### Configuration

- `backend/langgraph.json` — graph entry point (`kestrel_backend.graph.builder:build_discovery_graph`) and env reference
- `backend/.env` — API keys (not committed)

### Notes

- Studio is a dev tool only — does not affect production deployment
- Hot-reloads on code changes
- Runs in-memory (no Docker/Redis/Postgres needed)
- Port 2024 by default
- **Observability**: Langfuse handles tracing, not LangSmith. `LANGSMITH_API_KEY` is not needed — ignore the Studio banner about it.

## Architecture

Full-stack application: React frontend + Python FastAPI backend communicating via WebSocket.

```
client/              React 18 SPA (Vite, Wouter router, TanStack Query, shadcn/ui + Tailwind)
backend/             FastAPI backend with Claude Agent SDK + LangGraph discovery pipeline
  src/kestrel_backend/
    main.py          FastAPI app with WebSocket endpoints
    agent.py         Claude Agent SDK integration
    kestrel_client.py    Kestrel MCP client (biomedical knowledge graph)
    graph/           LangGraph discovery pipeline (9-node workflow)
      state.py       Pydantic models and TypedDict state
      builder.py     Graph construction and routing
      runner.py      Pipeline execution and streaming
      nodes/         Individual pipeline nodes
docs/                Architecture documentation
  solutions/         Documented solutions (bugs, best practices, patterns) with YAML frontmatter
```

### Dual Operating Modes

1. **Classic Mode** — Single Claude agent with Kestrel MCP tools for direct KG queries
2. **Discovery Pipeline Mode** — 9-node LangGraph workflow for research-grade analysis

### Discovery Pipeline Nodes

`Intake → Entity Resolution → Triage → [Direct KG | Cold-Start] → Pathway Enrichment → Integration → [Temporal] → Synthesis`

| Node | Purpose |
|------|---------|
| Intake | Query parsing, entity extraction (heuristic, no LLM) |
| Entity Resolution | CURIE mapping via Kestrel MCP |
| Triage | Edge count classification (well-characterized/moderate/sparse/cold-start) |
| Direct KG | Disease associations, pathway memberships for well-characterized entities |
| Cold-Start | Analogue-based inference for sparse/unknown entities |
| Pathway Enrichment | Shared neighbors, biological themes |
| Integration | Cross-entity bridges, gap analysis |
| Temporal | Longitudinal classification (conditional) |
| Synthesis | Final report + hypothesis generation |

### Key Technical Details

- **Concurrency**: `SDK_SEMAPHORE = asyncio.Semaphore(4)` controls parallel SDK calls
- **Buffer Size**: `max_buffer_size=10MB` for large KG responses
- **Hub Detection**: Nodes with >1000 edges flagged, deprioritized in analysis
- **Two-Tier Direct KG**: Tier 1 (top entities) + Tier 2 (discovered entities) analysis

## Development Workflow

### Feature Development

1. **Create feature branch** from `main`:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feat/your-feature-name
   ```

2. **Develop locally**, commit changes with descriptive messages

3. **Push and create PR**:
   ```bash
   git push -u origin feat/your-feature-name
   gh pr create --title "feat: description" --body "## Summary\n..."
   ```

4. **Wait for Greptile code review** — automated review runs on PR

5. **Address review feedback** if needed

6. **User merges PR** — merge to `main` triggers automatic deployment via GitHub Actions

### Branch Naming Conventions

| Prefix | Purpose |
|--------|---------|
| `feat/` | New features |
| `fix/` | Bug fixes |
| `refactor/` | Code refactoring |
| `perf/` | Performance improvements |
| `docs/` | Documentation updates |

## Production Deployment

### AWS Lightsail Instance

| Property | Value |
|----------|-------|
| Instance | `expert-in-the-loop-upgraded` |
| IP | `35.161.242.62` |
| URL | `https://kraken.expertintheloop.io` |
| Backend Port | 8000 |
| Service | `kraken-backend` |

### SSH Access

```bash
ssh -i ~/.ssh/lightsail-expert.pem ubuntu@35.161.242.62
```

### Service Management

```bash
# Check status
sudo systemctl status kraken-backend
sudo journalctl -u kraken-backend -f

# Restart service
sudo systemctl restart kraken-backend

# View recent logs
sudo journalctl -u kraken-backend --since "10 minutes ago"
```

### Manual Deployment (if needed)

```bash
cd ~/kraken-chatbot
git fetch origin main
git reset --hard origin/main

# Rebuild frontend with production WebSocket URL
npm ci
VITE_WS_URL=wss://kraken.expertintheloop.io/ws/chat npm run build

# Update backend dependencies
cd backend
~/.local/bin/uv sync

# Restart service
sudo systemctl restart kraken-backend

# Verify health
curl -sf http://localhost:8000/health
```

### Automated Deployment

Pushing to `main` triggers GitHub Actions workflow (`.github/workflows/deploy.yml`):
1. SSH into Lightsail
2. Pull latest code
3. Rebuild frontend with production WebSocket URL
4. Sync backend dependencies with uv
5. Restart `kraken-backend` service
6. Health check with retries

### Claude CLI Authentication

The backend uses Claude Agent SDK which authenticates via OAuth:

```bash
# On production server
claude login  # Opens browser for OAuth
```

If authentication expires, the backend returns `AUTH_ERROR` to the frontend.

## Dev Branch Workflow

### Overview

| Property | Value |
|----------|-------|
| Branch | `dev` (long-lived integration branch) |
| URL | `https://dev-kraken.expertintheloop.io` |
| Backend Port | 8006 |
| Service | `kraken-backend-dev` |
| Database | `kraken_dev` (separate from prod `kraken_db`) |
| Deploy Dir | `/home/ubuntu/kraken-chatbot-dev` |

### How It Works

Pushing to the `dev` branch triggers `.github/workflows/deploy-dev.yml`:
1. SSH into Lightsail
2. Pull latest code for the target branch
3. Rebuild frontend with `VITE_WS_URL=wss://dev-kraken.expertintheloop.io/ws/chat`
4. Sync backend dependencies with uv
5. Run Alembic migrations against `kraken_dev` database
6. Restart `kraken-backend-dev` service
7. Readiness check on port 8006

Manual dispatch: use `workflow_dispatch` to deploy any branch to the dev environment.

### Feature Branch Flow

1. Create feature branch from `dev`: `git checkout -b feat/my-feature`
2. Develop and push to feature branch
3. Merge into `dev` to test at `dev-kraken.expertintheloop.io`
4. Once verified, create PR from feature branch to `main` for production

### Dev Service Management

```bash
sudo systemctl status kraken-backend-dev
sudo journalctl -u kraken-backend-dev -f
sudo systemctl restart kraken-backend-dev
```

### Important Notes

- **Separate database**: Dev uses `kraken_dev`, prod uses `kraken_db`. Alembic migrations are independent.
- **Shared Claude OAuth tokens**: Prod and dev share `~/.claude/` on the server. If OAuth expires, both environments lose Claude access simultaneously. Run `claude login` on the server to refresh.
- **Concurrency group**: Dev deploys use `lightsail-deploy-kraken`. A separate PR to `main` should add the same group to `deploy.yml` for full dev-prod serialization. Until then, concurrent dev+prod deploys are possible but unlikely.
- **`origin/dev_main` branch**: Stale, superseded by `dev`. Do not use.

### Server-Side Setup (for new dev environment)

1. `git clone` the repo to `/home/ubuntu/kraken-chatbot-dev`, checkout `dev`
2. `createdb kraken_dev`
3. Copy `deploy/dev/.env.example` to `backend/.env`, fill in secrets, `chmod 600`
4. `sudo cp deploy/dev/kraken-backend-dev.service /etc/systemd/system/`
5. `sudo cp deploy/dev/nginx-kraken-dev.conf /etc/nginx/sites-available/`, symlink to `sites-enabled`
6. `sudo certbot --nginx -d dev-kraken.expertintheloop.io`
7. `sudo nginx -t && sudo systemctl reload nginx`
8. `sudo systemctl daemon-reload && sudo systemctl enable kraken-backend-dev`
9. Push any change to `dev` to trigger the first deploy

## Clerk Authentication

### How It Works

Users sign in via [Clerk](https://clerk.com). The frontend uses `@clerk/react` with a proxy URL. The backend verifies Clerk JWT tokens using PyJWT + JWKS.

```
Browser → ClerkProvider (proxyUrl → /api/__clerk) → Clerk FAPI
         → useAuth().getToken() → WebSocket ?token=<jwt>
         → FastAPI verifies JWT via PyJWKClient (RS256, JWKS)
         → Email domain whitelist check
```

### Environment Variables

| Variable | Where | Purpose |
|----------|-------|---------|
| `CLERK_AUTH_ENABLED` | Backend | `true` to enable auth. Fails to start if `true` without `CLERK_SECRET_KEY` |
| `CLERK_SECRET_KEY` | Backend | Clerk server secret for proxy and JWKS |
| `CLERK_JWKS_URL` | Backend | Clerk JWKS endpoint for token verification |
| `CLERK_ISSUER` | Backend | Expected JWT `iss` claim |
| `CLERK_PROXY_URL` | Backend | Full URL of the Clerk proxy (e.g., `https://dev-kraken.../api/__clerk`) |
| `ALLOWED_EMAIL_DOMAINS` | Backend | Comma-separated email domains (authoritative) |
| `ALLOWED_EMAILS` | Backend | Comma-separated email overrides |
| `VITE_CLERK_PUBLISHABLE_KEY` | Frontend | Clerk public key (baked at build time) |
| `VITE_CLERK_PROXY_URL` | Frontend | Clerk proxy URL (baked at build time) |
| `VITE_ALLOWED_EMAIL_DOMAINS` | Frontend | UX-level domain gate (not authoritative) |
| `VITE_ALLOWED_EMAILS` | Frontend | UX-level email overrides |

### Key Files

| File | Purpose |
|------|---------|
| `backend/src/kestrel_backend/clerk_auth.py` | JWT verification, FastAPI dependencies |
| `backend/src/kestrel_backend/clerk_proxy.py` | Clerk FAPI proxy at `/api/__clerk` |
| `client/src/components/ProtectedRoute.tsx` | Frontend auth gate + email domain check |
| `client/src/pages/Login.tsx` | Clerk SignIn page |
| `client/src/pages/AccessDenied.tsx` | Unauthorized email domain page |

### Local Development

Set `CLERK_AUTH_ENABLED=false` in `backend/.env` to disable auth entirely for local development. The frontend also skips Clerk when `VITE_CLERK_PUBLISHABLE_KEY` is not set.

## Recent Bug Fixes (Reference)

### PR #6: Duplicate CURIE Handling + Regex Fix

**Duplicate CURIEs in Direct KG** (`direct_kg.py`):
- Issue: When same CURIE appeared in both tier1 and tier2, code used `asyncio.sleep(0)` as placeholder which returned `None` instead of empty tuple
- Fix: Deduplicate CURIEs before creating async tasks, then map results back to all indices

**Alias Regex Pattern** (`intake.py`):
- Issue: `*?` quantifier could match zero characters
- Fix: Changed to `+?` to require at least one character

## Key Files for Common Tasks

| Task | Files |
|------|-------|
| Add new pipeline node | `graph/nodes/`, `graph/builder.py`, `graph/state.py` |
| Modify entity resolution | `graph/nodes/entity_resolution.py` |
| Adjust KG analysis | `graph/nodes/direct_kg.py`, `graph/nodes/cold_start.py` |
| Change WebSocket protocol | `backend/src/kestrel_backend/protocol.py`, `main.py` |
| Update state schema | `graph/state.py` |
| Frontend components | `client/src/` |

## Testing on Production

For testing changes on production before merge:

1. SSH into Lightsail
2. Checkout your feature branch directly:
   ```bash
   cd ~/kraken-chatbot
   git fetch origin
   git checkout feat/your-feature-name
   npm ci && VITE_WS_URL=wss://kraken.expertintheloop.io/ws/chat npm run build
   cd backend && uv sync
   sudo systemctl restart kraken-backend
   ```
3. Test at https://kraken.expertintheloop.io
4. **Important**: Reset to main after testing:
   ```bash
   git checkout main && git reset --hard origin/main
   sudo systemctl restart kraken-backend
   ```

---
> Source: [trentleslie/kraken-chatbot](https://github.com/trentleslie/kraken-chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
