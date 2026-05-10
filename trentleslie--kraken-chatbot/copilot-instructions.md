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
cd backend && uv run pytest tests/ -v -m "not integration"
```

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
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
