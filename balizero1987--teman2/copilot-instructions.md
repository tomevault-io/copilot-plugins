## teman2

> Production AI platform (v5.2.0) - FastAPI backend + Next.js frontend

# Nuzantara Project Rules for Cursor

## Project Type
Production AI platform (v5.2.0) - FastAPI backend + Next.js frontend

## Stack
- Backend: Python 3.11+, FastAPI, **88 routers**, **244 services**, 385 test files
- Frontend: Next.js (App Router), TypeScript, Tailwind CSS
- DB: PostgreSQL 17, Qdrant (9 collections, 66,595 vectors), Redis
- Deploy: Fly.io `nuzantara-rag` (backend, 2GB RAM, auto_stop), Vercel (frontend)
- Embedding: `text-embedding-3-small` (1536 dims) — **NEVER CHANGE, FROZEN**
- KG: LangGraph — 56,113 nodes, 161,173 edges
- MCP: 109 tools (nuzantara-mcp) + 14 tools (nuzantara-mcp-advanced)
- Channels: 7 (WhatsApp, Telegram, Instagram, X/Twitter, Web, Google Chat, Slack)

## Golden Rules (ENFORCE)

1. **Virtualenv mandatory** - Never system Python
2. **No root exec** - Use `PYTHONPATH=. python -m backend.module`
3. **Absolute imports** - `from backend.core import config`, never relative
4. **Async first** - `httpx` not `requests`, all I/O async
5. **Type hints required** - Every function fully typed
6. **No hardcoded secrets** - Use env vars
7. **Data/logic separation** - Clean architecture
8. **Logger not print()** - Use structured logging
9. **Quality required** - Tests + error handling mandatory
10. **Verify sources** - Never presume, always verify

## Code Style

### Python
```python
# Good
from typing import Optional
from backend.core.logging import logger
import httpx

async def fetch_data(id: str) -> Optional[dict]:
    try:
        async with httpx.AsyncClient() as client:
            result = await client.get(f"/api/{id}")
            logger.info(f"Fetch successful: {id}")
            return result.json()
    except Exception as e:
        logger.error(f"Fetch failed: {id}", exc_info=True)
        raise
```

### TypeScript
```typescript
// Good
interface DataResponse {
  id: string;
  name: string;
}

async function fetchData(id: string): Promise<DataResponse | null> {
  try {
    const res = await fetch(`/api/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    console.error('Fetch failed:', error);
    return null;
  }
}
```

## Critical Paths
- Backend: `apps/backend-rag/backend/app/{routers,services}/`
- Frontend: `apps/mouth/src/{app,components,lib}/`
- Tests: `apps/backend-rag/tests/`
- MCP RAG: `apps/nuzantara-mcp/`
- MCP Ops: `apps/nuzantara-mcp-advanced/`
- Entry Fly.io: `apps/backend-rag/backend/main_cloud.py`
- Config: `apps/backend-rag/backend/core/`

## Commands

### Backend
```bash
source venv/bin/activate
PYTHONPATH=. python -m uvicorn backend.main:app --reload
PYTHONPATH=. pytest tests/ -v
mypy backend/
ruff check backend/
```

### Frontend
```bash
npm run dev
npm run build
npm run test
```

## Domain Rules

### KBLI Data
- **FLAT payloads only** in Qdrant
- Fields: `code`, `title_id`, `title_en`, `description`, `category`, `section`
- ❌ NO nested structures

### Pricing
- **ONLY from PricingTool**
- Never hardcode or cache prices
- Reference: `PRICING_REFERENCE.md`, `VISA_TYPES_REFERENCE.md`

### Evidence Scoring
- < 0.15: ABSTAIN (refuse to answer)
- 0.15-0.60: CAUTIOUS (add disclaimer)
- > 0.60: NORMAL (confident)

## Deployment
- Backend: `fly deploy --app nuzantara-rag`
- Frontend: Auto-deploy to Vercel on git push

## Owner
- Codename: Zero
- Real name: PRIVATE (never reveal)
- Language: Italian with owner, client's language with others

## Avoid
❌ System Python
❌ `requests` library
❌ Nested Qdrant payloads
❌ Hardcoded prices
❌ `print()` statements
❌ Relative imports
❌ Blocking I/O
❌ Missing types

## Require
✅ Virtualenv active
✅ `httpx` for HTTP
✅ Flat Qdrant payloads
✅ PricingTool for prices
✅ `logger` for logging
✅ Absolute imports
✅ Async/await
✅ Full type hints
✅ Tests for new features

## LangGraph KG — Architecture

5 nodi core:
1. `understand_query` — intent + entity extraction
2. `resolve_entities` — fuzzy matching to KG entity_ids
3. `traverse_graph` — BFS traversal (max 3 hops)
4. `reason` — LLM analysis of relationship chains
5. `synthesize_workflow` — convert chains to executable workflows

4 subgraph di dominio: `company`, `visa`, `property`, `tax`

## Rogue AI — Pattern da bloccare

❌ Rimuovere `Any` da `typing` imports → crash runtime su tutti i router
❌ Cambiare `httpx` con `requests` → viola async rule
❌ Payload nested in Qdrant → KBLI search rotta
❌ `--workers 2` nel Dockerfile → OOM kill su Fly.io (2GB VM, deve restare 1)
❌ Import relativi → crash runtime

## Pre-Deploy Checklist

```bash
# 1. Verifica modifiche non intenzionali
git diff --name-only HEAD -- apps/backend-rag/backend/

# 2. Test import chain critico
cd apps/backend-rag && source .venv/bin/activate
python -c "from backend.app.dependencies import get_current_user; print('OK')"

# 3. Core tests (82 test, <15s)
PYTHONPATH=. pytest backend/tests/services/rag/test_kg_langgraph.py \
  backend/tests/services/rag/test_kg_subgraphs.py \
  backend/tests/services/rag/test_confidence.py -q

# 4. Deploy rolling (no downtime)
fly deploy --strategy rolling --app nuzantara-rag
```

**Test debt cleaned (2026-03-20):** 0 failed, 0 errors. Core tests 100% (KG 82/82, Channels 43/43, RAG 244/244).

**Core Guardian V3:** Autonomous code quality agent runs every 3h ($0 cost). Do NOT interfere with its worktree-isolated fixes in `apps/evaluator/core_guardian/`.

**Fly.io:** SOLO 3 app (nuzantara-rag 2GB, nuzantara-postgres 2GB, nuzantara-qdrant 2GB). Cost ~$35-40/mo.
**bali-intel-scraper:** runs ONLY locally on Pro via OpenClaw, NOT on Fly.io (destroyed 2026-03-14).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Balizero1987) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
