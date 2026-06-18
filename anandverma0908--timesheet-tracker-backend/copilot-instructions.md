## timesheet-tracker-backend

> > Read this file before every session. It has everything Claude needs to build the Trackly backend.

# Trackly — Claude Backend Context
> Read this file before every session. It has everything Claude needs to build the Trackly backend.
> Work through weeks in order. Complete Week 1 before starting Week 2.

---

## Project Overview

Trackly is an AI-powered Knowledge Management System replacing Jira + Confluence for 3SC Solutions.
46 engineers · 8 PODs · Local hosting · Built-in AI called NOVA (no external APIs).

**Tagline:** The work OS for modern teams.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python 3.12) + Uvicorn |
| Database | PostgreSQL 15 + pgvector extension |
| ORM | SQLAlchemy 2.0 |
| Migrations | Alembic |
| AI Engine | NOVA — Llama 3.1 8B via Ollama (`http://ollama:11434`) |
| Embeddings | sentence-transformers `all-MiniLM-L6-v2` (384-dim, local) |
| Reranker | cross-encoder/ms-marco-MiniLM-L-6-v2 (local) |
| Auth | JWT HS256 24h + bcrypt 12 rounds |
| Scheduler | APScheduler |
| Hosting | Docker Compose (fully local) |

---

## Coding Conventions — Follow Always

- **Async routes** — all FastAPI routes are `async def`
- **No raw SQL strings** — use SQLAlchemy ORM or `text()` with bound params
- **Auth on every route** — `Depends(get_current_user)` — no exceptions
- **Org scoping** — every DB query filters by `org_id` — no cross-tenant leakage
- **Pydantic v2** — request body = `...Request`, response = `...Response`
- **Error handling** — raise `HTTPException(status_code=..., detail="...")`
- **Soft deletes** — use `is_deleted = True`, never hard DELETE on tickets/comments
- **Parameterised queries** — never interpolate user input into SQL strings

---

## Environment Variables

```env
DATABASE_URL=postgresql://trackly:trackly@postgres:5432/trackly
JWT_SECRET=<openssl rand -hex 32>
EMBEDDING_MODEL=all-MiniLM-L6-v2
NOVA_MODEL=llama3.1:8b
NOVA_BASE_URL=http://ollama:11434
NOVA_temperature=0
NOVA_MAX_TOKENS=1500
SMTP_HOST=mailhog
SMTP_PORT=1025
```

---

## Already Built — Do Not Rebuild

| File | What it does |
|---|---|
| `main.py` | FastAPI app setup, route registration, CORS |
| `auth.py` | JWT login, `get_current_user`, `get_admin` dependencies |
| `database.py` | SQLAlchemy engine, session, pgvector setup |
| `models.py` | Pydantic models for existing endpoints |

### Existing DB Tables
- `organisations` — org config (`id`, `name`, `jira_url`, `jira_token`)
- `users` — auth + hierarchy (`id`, `name`, `email`, `role`, `pod`, `emp_no`, `reporting_to`, `password_hash`, `org_id`)
- `jira_tickets` — synced tickets (`id`, `org_id`, `jira_key`, `summary`, `assignee`, `pod`, `client`, `hours_spent`, `status`, `issue_type`, `priority`)
- `worklogs` — time entries (`id`, `ticket_id`, `author`, `author_email`, `hours`, `log_date`)
- `manual_entries` — manual time logs (`id`, `user_id`, `org_id`, `activity`, `hours`, `entry_date`, `pod`, `client`, `status`)
- `ticket_embeddings` — pgvector (`id`, `ticket_id`, `embedding vector(384)`, `content_snippet`, `updated_at`)

### Existing API Endpoints
```
POST /api/auth/login
GET  /api/auth/me
POST /api/auth/set-password
GET  /api/summary
GET  /api/tickets
GET  /api/activity
GET  /api/filters
GET  /api/team
GET  /api/engineer-stats
POST /api/manual-entries
GET  /api/export/monthly
GET  /api/export/fy
GET  /api/users            (admin only)
POST /api/employees/sync
```

---

## Role Permissions

| Role | Tickets | Wiki | Reports | Admin |
|---|---|---|---|---|
| admin | Full CRUD | Full CRUD | All | Full |
| engineering_manager | Full CRUD own PODs | Full CRUD | Own PODs | Read |
| tech_lead | Full CRUD own PODs | Create/Edit | Own POD | None |
| team_member | Create + own | Create/Edit | Own only | None |
| finance_viewer | Read only | Read only | All read | None |

---

---

# ✅ WEEK 1 — NOVA Core + Ticket CRUD + AI Analysis
> **Deadline: April 17** | Complete these in order — each step unblocks the next.

## What to build this week

```
backend/
├── ai/
│   ├── nova.py                ← START HERE (everything depends on this)
│   ├── ticket_intelligence.py ← Step 2
│   └── search.py              ← Step 3
├── routes/
│   └── tickets.py             ← Step 4
└── alembic/versions/
    └── xxx_add_ticket_tables.py  ← Step 5 (comments + attachments)
```

---

## Step 1 — `backend/ai/nova.py` (NOVA Core Engine)

Build this first. Every AI feature in the project calls this file.

```python
"""
NOVA — Neural Orchestration & Velocity Assistant
Trackly's built-in AI. 100% local. Zero external API.
"""
import httpx
import json
from sentence_transformers import SentenceTransformer, CrossEncoder
from typing import Optional
import logging

logger = logging.getLogger(__name__)

EMBEDDING_MODEL = SentenceTransformer("all-MiniLM-L6-v2")
RERANKER_MODEL  = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
OLLAMA_BASE_URL = "http://ollama:11434"
OLLAMA_MODEL    = "llama3.1:8b"

NOVA_SYSTEM_PROMPT = """You are NOVA, the built-in AI assistant for Trackly —
a work management platform used by engineering and cross-functional teams at 3SC Solutions.
Be concise, accurate, and helpful. When analysing tickets be specific.
When generating documents use markdown formatting.
Always ground your answers in the provided context when available."""


async def chat(
    user_message: str,
    system_prompt: Optional[str] = None,
    context_docs: Optional[list[str]] = None,
    temperature: float = 0.3,
    max_tokens: int = 1500,
) -> str:
    system = system_prompt or NOVA_SYSTEM_PROMPT
    if context_docs:
        context_block = "\n\n---\n\n".join(context_docs[:5])
        system += f"\n\n## Relevant context:\n\n{context_block}"
    messages = [
        {"role": "system", "content": system},
        {"role": "user",   "content": user_message},
    ]
    async with httpx.AsyncClient(timeout=60.0) as client:
        resp = await client.post(
            f"{OLLAMA_BASE_URL}/api/chat",
            json={"model": OLLAMA_MODEL, "messages": messages, "stream": False,
                  "options": {"temperature": temperature, "num_predict": max_tokens}}
        )
        resp.raise_for_status()
        return resp.json()["message"]["content"]


def embed(text: str) -> list[float]:
    return EMBEDDING_MODEL.encode(text, normalize_embeddings=True).tolist()


def embed_batch(texts: list[str]) -> list[list[float]]:
    return EMBEDDING_MODEL.encode(
        texts, normalize_embeddings=True, batch_size=32
    ).tolist()


def rerank(query: str, documents: list[str], top_k: int = 5) -> list[int]:
    pairs  = [(query, doc) for doc in documents]
    scores = RERANKER_MODEL.predict(pairs)
    ranked = sorted(range(len(scores)), key=lambda i: scores[i], reverse=True)
    return ranked[:top_k]


def is_available() -> bool:
    try:
        import httpx as _httpx
        resp   = _httpx.get(f"{OLLAMA_BASE_URL}/api/tags", timeout=3.0)
        models = [m["name"] for m in resp.json().get("models", [])]
        return any(OLLAMA_MODEL.split(":")[0] in m for m in models)
    except Exception:
        return False
```

---

## Step 2 — `backend/ai/ticket_intelligence.py`

```python
"""NOVA Ticket Intelligence — agentic flow for smart ticket creation."""
import json, asyncio
from .nova import chat, embed
from .search import find_similar_tickets

TICKET_CLASSIFY_PROMPT = """Analyse this ticket and extract structured information.
Return ONLY valid JSON, no other text.

Ticket: {text}

Return JSON with these exact fields:
{{
  "title": "concise ticket title (max 100 chars)",
  "description": "expanded description with more detail",
  "issue_type": "Bug|Story|Task|Epic",
  "priority": "Critical|High|Medium|Low",
  "pod": "one of: DPAI, EDM, SNOP, SNOE, PA, IAM, PLAT, SNPRM, TMSNG",
  "client": "client name or null",
  "story_points": 1,
  "labels": ["label1"],
  "suggested_assignee_role": "SDE1|SDE2|SDE3|SDET|BA",
  "confidence": 0.0,
  "reasoning": "brief explanation"
}}"""


async def analyse_ticket(text: str) -> dict:
    """Step 1 — extract structured fields from NL text via NOVA."""
    try:
        raw   = await chat(TICKET_CLASSIFY_PROMPT.format(text=text), temperature=0, max_tokens=500)
        start = raw.find("{")
        end   = raw.rfind("}") + 1
        return json.loads(raw[start:end])
    except Exception as e:
        return {"error": str(e), "title": text[:100]}


async def full_analysis(nl_text: str, org_id: str) -> dict:
    """Full agentic pipeline: classify + duplicate check in parallel."""
    init_embed = embed(nl_text)
    fields_task = analyse_ticket(nl_text)
    dupes_task  = find_similar_tickets(init_embed, org_id, threshold=0.85, limit=3)
    fields, dupes = await asyncio.gather(fields_task, dupes_task)
    return {
        "fields":          fields,
        "duplicates":      dupes,
        "has_duplicates":  len(dupes) > 0,
    }
```

---

## Step 3 — `backend/ai/search.py`

```python
"""NOVA Search — semantic search + RAG pipeline."""
from .nova import embed, rerank, chat
from database import get_db
from sqlalchemy import text

SEARCH_SYSTEM = """You are NOVA answering questions about Trackly work data.
Use ONLY the provided context. If the answer is not in context, say so.
Be concise. Cite sources by name."""


async def semantic_search(query: str, org_id: str, limit: int = 10) -> list[dict]:
    """Search tickets + wiki pages by semantic similarity."""
    query_emb = str(embed(query))
    db = next(get_db())

    tickets = db.execute(text("""
        SELECT 'ticket' as source_type, t.jira_key as key, t.summary as title,
               te.content_snippet as snippet,
               1 - (te.embedding <=> :emb::vector) as similarity
        FROM ticket_embeddings te
        JOIN jira_tickets t ON te.ticket_id = t.id
        WHERE t.org_id = :org_id
        ORDER BY te.embedding <=> :emb::vector LIMIT :limit
    """), {"emb": query_emb, "org_id": org_id, "limit": limit}).fetchall()

    wiki = db.execute(text("""
        SELECT 'wiki' as source_type, wp.id::text as key, wp.title,
               we.content_snippet as snippet,
               1 - (we.embedding <=> :emb::vector) as similarity
        FROM wiki_embeddings we
        JOIN wiki_pages wp ON we.page_id = wp.id
        WHERE wp.org_id = :org_id
        ORDER BY we.embedding <=> :emb::vector LIMIT :limit
    """), {"emb": query_emb, "org_id": org_id, "limit": limit}).fetchall()

    all_results = [dict(r) for r in list(tickets) + list(wiki)]
    if not all_results:
        return []

    snippets    = [r["snippet"] or r["title"] for r in all_results]
    top_indices = rerank(query, snippets, top_k=min(8, len(all_results)))
    return [all_results[i] for i in top_indices]


async def nl_query(query: str, org_id: str) -> dict:
    """RAG: search → retrieve → NOVA synthesises answer + citations."""
    results  = await semantic_search(query, org_id, limit=5)
    contexts = [f"[{r['source_type'].upper()}] {r['title']}\n{r['snippet']}" for r in results]
    answer   = await chat(
        user_message=f"Question: {query}\n\nAnswer based on the context provided:",
        context_docs=contexts,
        temperature=0,
    )
    return {"answer": answer, "sources": results}


async def find_similar_tickets(
    embedding: list[float], org_id: str,
    threshold: float = 0.85, limit: int = 3
) -> list[dict]:
    """Find open tickets similar to given embedding — used for duplicate detection."""
    db  = next(get_db())
    emb = str(embedding)
    rows = db.execute(text("""
        SELECT t.jira_key, t.summary, t.status,
               1 - (te.embedding <=> :emb::vector) as similarity
        FROM ticket_embeddings te
        JOIN jira_tickets t ON te.ticket_id = t.id
        WHERE t.org_id = :org_id
          AND t.status != 'Done'
          AND 1 - (te.embedding <=> :emb::vector) >= :threshold
        ORDER BY te.embedding <=> :emb::vector LIMIT :limit
    """), {"emb": emb, "org_id": org_id, "threshold": threshold, "limit": limit}).fetchall()
    return [dict(r) for r in rows]


async def embed_and_store_ticket(ticket_id: str, title: str, description: str, db) -> None:
    """Generate embedding for a ticket and upsert into ticket_embeddings."""
    from .nova import embed as nova_embed
    content   = f"{title}\n{description or ''}"
    embedding = str(nova_embed(content))
    snippet   = content[:500]
    db.execute(text("""
        INSERT INTO ticket_embeddings (id, ticket_id, embedding, content_snippet, updated_at)
        VALUES (gen_random_uuid(), :tid, :emb::vector, :snippet, NOW())
        ON CONFLICT (ticket_id) DO UPDATE
        SET embedding = :emb::vector, content_snippet = :snippet, updated_at = NOW()
    """), {"tid": ticket_id, "emb": embedding, "snippet": snippet})
    db.commit()


async def embed_and_store_wiki(page_id: str, title: str, content_md: str, db) -> None:
    """Generate embedding for a wiki page and upsert into wiki_embeddings."""
    from .nova import embed as nova_embed
    content   = f"{title}\n{content_md or ''}"
    embedding = str(nova_embed(content))
    snippet   = content[:500]
    db.execute(text("""
        INSERT INTO wiki_embeddings (id, page_id, embedding, content_snippet, updated_at)
        VALUES (gen_random_uuid(), :pid, :emb::vector, :snippet, NOW())
        ON CONFLICT (page_id) DO UPDATE
        SET embedding = :emb::vector, content_snippet = :snippet, updated_at = NOW()
    """), {"pid": page_id, "emb": embedding, "snippet": snippet})
    db.commit()
```

---

## Step 4 — `backend/routes/tickets.py`

Build these endpoints in this order:

| # | Endpoint | Priority |
|---|---|---|
| BE-1.1 | POST /api/tickets | P0 |
| BE-1.1 | GET /api/tickets (extend existing) | P0 |
| BE-1.1 | PUT /api/tickets/:id | P0 |
| BE-1.1 | DELETE /api/tickets/:id (soft) | P0 |
| BE-1.2 | POST /api/tickets/nl-create | P0 |
| BE-1.3 | POST /api/tickets/ai-analyze | P0 |
| BE-1.5 | Status transition + audit log | P0 |
| BE-1.9 | Auto-embed on create/update | P0 |
| BE-1.6 | POST/GET /api/tickets/:id/comments | P1 |
| BE-1.7 | POST /api/tickets/:id/attachments | P1 |
| BE-1.8 | GET /api/tickets/:id/activity | P1 |

Key requirements:
- All endpoints use `Depends(get_current_user)` — no public routes
- Ticket create/update triggers `embed_and_store_ticket()` asynchronously
- NL create calls `full_analysis()` from `ticket_intelligence.py`
- Status transitions log to `audit_log` table
- Soft delete: set `is_deleted = True`, never hard delete

---

## Step 5 — Alembic Migration (DB-5): Comments + Attachments

```sql
CREATE TABLE ticket_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id UUID REFERENCES jira_tickets(id) ON DELETE CASCADE,
  author_id UUID REFERENCES users(id),
  body TEXT NOT NULL,
  parent_id UUID REFERENCES ticket_comments(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  is_deleted BOOLEAN DEFAULT false
);

CREATE TABLE ticket_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id UUID REFERENCES jira_tickets(id) ON DELETE CASCADE,
  filename TEXT NOT NULL,
  filepath TEXT NOT NULL,
  size_bytes INTEGER,
  uploaded_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Week 1 — NOVA Routes (`backend/routes/nova.py`)

Also add these NOVA endpoints this week (they're needed for the ticket AI features):

```python
GET  /api/nova/status     # check if Ollama is running
POST /api/nova/query      # NL query RAG — used by search modal
POST /api/tickets/nl-create    # calls full_analysis()
POST /api/tickets/ai-analyze   # calls full_analysis()
```

---

## Week 1 Dependencies to Install

```txt
ollama==0.3.3
sentence-transformers==3.0.1
torch==2.4.0
scikit-learn==1.5.1
numpy==1.26.4
httpx==0.27.2
apscheduler==3.10.4
python-multipart==0.0.12
```

---

## Week 1 Done Checklist

- [ ] `backend/ai/nova.py` — chat(), embed(), rerank(), is_available()
- [ ] `backend/ai/ticket_intelligence.py` — analyse_ticket(), full_analysis()
- [ ] `backend/ai/search.py` — semantic_search(), nl_query(), find_similar_tickets(), embed helpers
- [ ] `backend/routes/tickets.py` — full CRUD + NL create + AI analyze + comments + attachments
- [ ] `backend/routes/nova.py` — /api/nova/status, /api/nova/query
- [ ] Alembic migration — ticket_comments, ticket_attachments tables
- [ ] Auto-embed runs on ticket create/update
- [ ] Duplicate detection works (pgvector similarity > 0.85)
- [ ] Test: POST /api/tickets/nl-create with plain English sentence

---

---

# ⬜ WEEK 2 — Wiki + Semantic Search + Seed Data
> **Deadline: April 17** | Start only after Week 1 checklist is complete.

## What to build this week

```
backend/
├── ai/
│   └── (search.py already has wiki embedding helpers from Week 1)
├── routes/
│   ├── wiki.py        ← wiki spaces + pages + versions
│   └── search.py      ← /api/search endpoint
└── seeds/
    ├── seed_tickets.py
    ├── seed_wiki.py
    └── seed_all.py
alembic/versions/
    └── xxx_add_wiki_tables.py   ← DB-3
```

---

## DB Migration (DB-3) — Wiki Tables

```sql
CREATE TABLE wiki_spaces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organisations(id),
  name TEXT NOT NULL,
  slug TEXT NOT NULL,
  description TEXT,
  access_level TEXT DEFAULT 'private',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE wiki_pages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  space_id UUID REFERENCES wiki_spaces(id) ON DELETE CASCADE,
  parent_id UUID REFERENCES wiki_pages(id),
  org_id UUID REFERENCES organisations(id),
  title TEXT NOT NULL,
  content_md TEXT,
  content_html TEXT,
  version INTEGER DEFAULT 1,
  author_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE wiki_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  page_id UUID REFERENCES wiki_pages(id) ON DELETE CASCADE,
  version INTEGER NOT NULL,
  content_md TEXT,
  author_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE wiki_embeddings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  page_id UUID REFERENCES wiki_pages(id) ON DELETE CASCADE,
  embedding vector(384),
  content_snippet TEXT,
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(page_id)
);
```

---

## Week 2 Endpoints to Build

### Wiki (`backend/routes/wiki.py`)

| Endpoint | Method | Auth | Notes |
|---|---|---|---|
| /api/wiki/spaces | GET | JWT | List spaces accessible to user |
| /api/wiki/spaces | POST | PM+ | Create space |
| /api/wiki/spaces/:id | PUT | PM+ | Update space |
| /api/wiki/spaces/:id | DELETE | Admin | Delete space |
| /api/wiki/pages | GET | JWT | List pages in a space (query param: space_id) |
| /api/wiki/pages | POST | Dev+ | Create page — auto-embed on save |
| /api/wiki/pages/:id | GET | JWT | Get page with content |
| /api/wiki/pages/:id | PUT | Dev+ | Update page — bump version, save to wiki_versions, re-embed |
| /api/wiki/pages/:id/versions | GET | JWT | List version history |
| /api/wiki/pages/:id/restore | POST | PM+ | Restore a version |
| /api/wiki/pages/:id/related | GET | JWT | Top 5 similar pages (pgvector) |
| /api/wiki/ai/meeting-notes | POST | Dev+ | Extract action items from raw notes |

### Search (`backend/routes/search.py`)

| Endpoint | Method | Notes |
|---|---|---|
| /api/search | POST | Body: `{ query, scope: "all"|"tickets"|"wiki" }` — returns reranked results |

---

## Week 2 — Seed Data

### `backend/seeds/seed_tickets.py`
Create all 35 tickets in exact order below. After inserting each ticket, call `embed_and_store_ticket()`.

**DPAI Project (20 tickets)**
```python
tickets = [
  {"jira_key":"DPAI-1018","summary":"Forecasting UI — drag & drop column reorder","issue_type":"Story","status":"In Progress","assignee":"Anand Verma","pod":"DPAI","priority":"High","story_points":8},
  {"jira_key":"DPAI-1015","summary":"JFL UAT environment slow to load (DFU drawer)","issue_type":"Bug","status":"In Progress","assignee":"Mohit Kapoor","pod":"DPAI","priority":"High","story_points":5},
  {"jira_key":"DPAI-1012","summary":"Group by pagination fails on 2nd page","issue_type":"Bug","status":"In Review","assignee":"Prakash Kumar","pod":"DPAI","priority":"Medium","story_points":3},
  {"jira_key":"DPAI-1010","summary":"Analytics dashboard redesign","issue_type":"Epic","status":"In Review","assignee":"Anand Verma","pod":"DPAI","priority":"High","story_points":13},
  {"jira_key":"DPAI-1009","summary":"Login page redesign with password strength","issue_type":"Task","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"Low","story_points":5},
  {"jira_key":"DPAI-7018","summary":"Data grid not visible until scroll on QA env","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"High","story_points":3},
  {"jira_key":"DPAI-7013","summary":"Drag & drop and hide column not working","issue_type":"Bug","status":"Done","assignee":"Prakash Kumar","pod":"DPAI","priority":"High","story_points":3},
  {"jira_key":"DPAI-7011","summary":"Unnecessary API call on group by 2nd page","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"Medium","story_points":2},
  {"jira_key":"DPAI-7000","summary":"Data grid display incorrect — QA Forecasting","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"High","story_points":3},
  {"jira_key":"DPAI-6972","summary":"Rearrangement broken in manage column","issue_type":"Bug","status":"Done","assignee":"Prakash Kumar","pod":"DPAI","priority":"Medium","story_points":2},
  {"jira_key":"DPAI-6971","summary":"Manage column dropdowns editable in QA","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"Low","story_points":1},
  {"jira_key":"DPAI-6856","summary":"JFL UAT — DFU info side drawer lag","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"DPAI","priority":"High","story_points":3},
  {"jira_key":"DPAI-1021","summary":"Refactor auth middleware for better performance","issue_type":"Story","status":"Backlog","assignee":"Anand Verma","pod":"DPAI","priority":"Medium","story_points":5},
  {"jira_key":"DPAI-1022","summary":"Add rate limiting to public APIs","issue_type":"Task","status":"Backlog","assignee":"Prakash Kumar","pod":"DPAI","priority":"Medium","story_points":3},
  {"jira_key":"DPAI-1019","summary":"Mobile responsive fixes for data grid","issue_type":"Bug","status":"Backlog","assignee":"Achal Kokatanoor","pod":"DPAI","priority":"Low","story_points":3},
  {"jira_key":"DPAI-1020","summary":"Dark mode support for forecasting screens","issue_type":"Story","status":"Backlog","assignee":"Mohit Kapoor","pod":"DPAI","priority":"Low","story_points":5},
  {"jira_key":"DPAI-1023","summary":"Export forecasting data to Excel","issue_type":"Task","status":"Backlog","assignee":"Swapnil Akash","pod":"DPAI","priority":"Low","story_points":2},
  {"jira_key":"DPAI-1024","summary":"Performance profiling — identify slow queries","issue_type":"Task","status":"Backlog","assignee":"Prakash Kumar","pod":"DPAI","priority":"Medium","story_points":3},
  {"jira_key":"DPAI-1025","summary":"Add Jest unit tests for grid component","issue_type":"Task","status":"Backlog","assignee":"Anand Verma","pod":"DPAI","priority":"Low","story_points":3},
  {"jira_key":"DPAI-1026","summary":"API documentation — Swagger cleanup","issue_type":"Task","status":"Backlog","assignee":"Mohit Kapoor","pod":"DPAI","priority":"Low","story_points":1},
]
```

**SNOP Project (15 tickets)**
```python
tickets += [
  {"jira_key":"SNOP-138","summary":"Edit button overlapping in Turkish language","issue_type":"Bug","status":"In Review","assignee":"Anand Verma","pod":"SNOP","priority":"Low","story_points":2},
  {"jira_key":"SNOP-139","summary":"SLA dashboard — overdue alerts not firing","issue_type":"Bug","status":"In Progress","assignee":"Aman Kumar Singh","pod":"SNOP","priority":"High","story_points":3},
  {"jira_key":"SNOP-140","summary":"Sprint capacity planning view","issue_type":"Story","status":"In Progress","assignee":"Akash Kumar","pod":"SNOP","priority":"Medium","story_points":5},
  {"jira_key":"SNOP-141","summary":"Vendor scorecard integration","issue_type":"Epic","status":"Backlog","assignee":"Ishu Rani","pod":"SNOP","priority":"Medium","story_points":13},
  {"jira_key":"SNOP-108","summary":"Filter panel overlapping grid on demand sensing","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"SNOP","priority":"High","story_points":2},
  {"jira_key":"SNOP-109","summary":"Blank page on pagination rows > 11","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"SNOP","priority":"High","story_points":2},
  {"jira_key":"SNOP-113","summary":"Something Went Wrong error in demand sensing window","issue_type":"Bug","status":"Done","assignee":"Anand Verma","pod":"SNOP","priority":"High","story_points":3},
  {"jira_key":"SNOP-142","summary":"Turkish locale — number formatting edge cases","issue_type":"Bug","status":"Backlog","assignee":"Akanksha","pod":"SNOP","priority":"Low","story_points":2},
  {"jira_key":"SNOP-143","summary":"Export supply plan to PDF","issue_type":"Task","status":"Backlog","assignee":"Ashish Kumar Gopalika","pod":"SNOP","priority":"Low","story_points":2},
  {"jira_key":"SNOP-144","summary":"Integrate with ERP system via REST","issue_type":"Epic","status":"Backlog","assignee":"Anoop Kumar Rai","pod":"SNOP","priority":"Medium","story_points":13},
  {"jira_key":"SNOP-145","summary":"Bulk import supplier data from CSV","issue_type":"Story","status":"Backlog","assignee":"Aastha Rai","pod":"SNOP","priority":"Medium","story_points":5},
  {"jira_key":"SNOP-146","summary":"Real-time stock level dashboard","issue_type":"Story","status":"Backlog","assignee":"Akash Kumar","pod":"SNOP","priority":"Medium","story_points":5},
  {"jira_key":"SNOP-147","summary":"AI demand forecasting model — backtest","issue_type":"Epic","status":"Backlog","assignee":"Prakash Kumar","pod":"SNOP","priority":"High","story_points":13},
  {"jira_key":"SNOP-148","summary":"Notification system — low stock alerts","issue_type":"Story","status":"Backlog","assignee":"Mohit Kapoor","pod":"SNOP","priority":"Medium","story_points":3},
  {"jira_key":"SNOP-149","summary":"Role-based supply chain view per region","issue_type":"Story","status":"Backlog","assignee":"Aman Kumar Singh","pod":"SNOP","priority":"Medium","story_points":5},
]
```

### `backend/seeds/seed_wiki.py`
Create these 12 wiki pages with realistic content. After each page, call `embed_and_store_wiki()`.

```python
pages = [
  {"space":"Engineering","title":"API Versioning Strategy","type":"ADR"},
  {"space":"Engineering","title":"Auth Middleware Architecture","type":"Runbook"},
  {"space":"Engineering","title":"Frontend Architecture Guidelines","type":"Standards"},
  {"space":"Engineering","title":"Database Schema Reference","type":"Reference"},
  {"space":"Engineering","title":"Rate Limiting Policy","type":"Decision"},
  {"space":"DPAI","title":"Forecasting Module Technical Design","type":"PRD"},
  {"space":"DPAI","title":"JFL UAT Sprint 14 Retrospective","type":"Retro"},
  {"space":"DPAI","title":"DFU Side Drawer Performance Investigation","type":"Incident"},
  {"space":"SNOP","title":"Turkish Localisation Checklist","type":"Checklist"},
  {"space":"SNOP","title":"Supply Chain Data Model","type":"Reference"},
  {"space":"Processes","title":"Engineering On-Call Runbook","type":"Process"},
  {"space":"Processes","title":"Code Review Standards","type":"Process"},
]
```

Generate realistic Markdown content for each page — at least 300 words, relevant to a 3SC engineering team.

### `backend/seeds/seed_all.py`
```python
# Run in order:
# 1. seed_tickets.py — inserts 35 tickets + embeddings
# 2. seed_wiki.py — inserts 12 wiki pages + embeddings
# Command: python seeds/seed_all.py
```

---

## Week 2 Done Checklist

- [ ] Alembic migration — wiki_spaces, wiki_pages, wiki_versions, wiki_embeddings
- [ ] `backend/routes/wiki.py` — all wiki CRUD endpoints
- [ ] Auto-embed on wiki page create/update
- [ ] `backend/routes/search.py` — POST /api/search
- [ ] `backend/seeds/seed_all.py` — 35 tickets + 12 wiki pages seeded with embeddings
- [ ] Test: POST /api/search with "rate limiting" → finds "Rate Limiting Policy" page

---

---

# ⬜ WEEK 3 — Sprint + NOVA Features + Analytics
> **Deadline: April 24** | Start only after Week 2 checklist is complete.

## What to build this week

```
backend/
├── ai/
│   ├── documents.py       ← retro, release notes, standup, meeting→actions
│   └── knowledge_gaps.py  ← gap detection weekly job
├── routes/
│   ├── sprints.py         ← sprint lifecycle
│   └── nova.py            ← extend with retro, standup, gaps endpoints
└── jobs/
    ├── scheduler.py        ← APScheduler setup
    ├── standup_job.py      ← 9AM daily
    ├── burnrate_job.py     ← hourly (stub — full build Week 4)
    └── gaps_job.py         ← weekly
alembic/versions/
    ├── xxx_add_sprints.py  ← DB-4
    └── xxx_add_standups.py ← DB-9
```

---

## DB Migrations

**DB-4 — Sprints**
```sql
CREATE TABLE sprints (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organisations(id),
  project_id UUID,
  name TEXT NOT NULL,
  goal TEXT,
  start_date DATE,
  end_date DATE,
  status TEXT DEFAULT 'planning',
  velocity INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);
ALTER TABLE jira_tickets ADD COLUMN sprint_id UUID REFERENCES sprints(id);
ALTER TABLE jira_tickets ADD COLUMN story_points INTEGER;
```

**DB-9 — Standups**
```sql
CREATE TABLE standups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  org_id UUID REFERENCES organisations(id),
  date DATE NOT NULL,
  yesterday TEXT,
  today TEXT,
  blockers TEXT,
  is_shared BOOLEAN DEFAULT false,
  generated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, date)
);
```

---

## Week 3 Endpoints to Build

### Sprints (`backend/routes/sprints.py`)

| Endpoint | Method | Notes |
|---|---|---|
| /api/sprints | GET | List sprints for org, filter by project |
| /api/sprints | POST | Create sprint (PM+) |
| /api/sprints/:id/start | POST | Start sprint — sets status=active, notifies team |
| /api/sprints/:id/complete | POST | Complete — moves unfinished to backlog, triggers retro |
| /api/sprints/:id/burndown | GET | Returns daily {date, ideal, actual} points |
| /api/sprints/:id/velocity | GET | Returns [{sprint_name, points_completed}] history |

### NOVA new endpoints (`backend/routes/nova.py` — extend)

| Endpoint | Method | Notes |
|---|---|---|
| /api/nova/sprint-retro/:id | POST | Generate sprint retro — calls documents.py |
| /api/nova/release-notes/:id | POST | Generate release notes |
| /api/nova/standup/generate | POST | Generate standup for user |
| /api/nova/standup/today | GET | Get own standup for today |
| /api/nova/standup/team | GET | Get all team standups (manager+) |
| /api/nova/standup/:id | PUT | Engineer edits own standup |
| /api/nova/knowledge-gaps | GET | List detected gaps (PM+) |

---

## `backend/ai/documents.py`

Build these 4 functions:

```python
async def generate_sprint_retro(sprint_id: str, org_id: str) -> str:
    # Fetch Done tickets from sprint → NOVA generates structured retro markdown

async def generate_release_notes(sprint_id: str, org_id: str) -> str:
    # Fetch Done tickets → NOVA generates grouped changelog markdown

async def extract_action_items(meeting_notes: str) -> list[dict]:
    # NOVA extracts: [{ action, owner, due, priority }] from raw notes text

async def generate_standup(user_id: str, org_id: str, date: str) -> dict:
    # Fetch yesterday worklogs + today In Progress tickets
    # NOVA generates Yesterday / Today / Blockers format
    # Saves to standups table, returns standup dict
```

---

## `backend/ai/knowledge_gaps.py`

```python
async def detect_knowledge_gaps(org_id: str) -> list[dict]:
    # 1. Fetch recent ticket titles (last 30 days)
    # 2. TF-IDF vectorize titles
    # 3. KMeans cluster into 8 topic groups
    # 4. Get top 3 terms per cluster as topic label
    # 5. semantic_search(topic_label) — check wiki coverage
    # 6. If best similarity < 0.70 → flag as gap
    # Returns: [{ topic, ticket_count, wiki_coverage_pct, example_tickets, suggestion }]
```

---

## `backend/jobs/scheduler.py`

```python
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()

def start_scheduler():
    scheduler.add_job(standup_job,   'cron',    hour=9, minute=0)  # 9AM daily Mon-Fri
    scheduler.add_job(burnrate_job,  'interval', hours=1)           # every hour
    scheduler.add_job(gaps_job,      'cron',    day_of_week='mon', hour=8)  # weekly Monday
    scheduler.start()
```

Register `start_scheduler()` in `main.py` on app startup.

---

## Week 3 Done Checklist

- [ ] Alembic migrations — sprints, standups tables
- [ ] `backend/routes/sprints.py` — full sprint lifecycle
- [ ] `backend/ai/documents.py` — retro, release notes, action items, standup
- [ ] `backend/ai/knowledge_gaps.py` — gap detection
- [ ] `backend/routes/nova.py` — retro, release notes, standup, gaps endpoints
- [ ] `backend/jobs/scheduler.py` — APScheduler setup + 3 jobs registered
- [ ] Test: POST /api/nova/sprint-retro/:id → returns markdown retro
- [ ] Test: POST /api/nova/standup/generate → returns standup for a user

---

---

# ⬜ WEEK 4 — Notifications + Burn Rate + Polish
> **Deadline: May 1** | Start only after Week 3 checklist is complete.

## What to build this week

```
backend/
├── routes/
│   ├── notifications.py   ← notification CRUD
│   └── clients.py         ← burn rate + budgets
└── jobs/
    └── burnrate_job.py    ← complete the hourly burn rate job
alembic/versions/
    ├── xxx_add_notifications.py  ← DB-6
    ├── xxx_add_audit_log.py      ← DB-7
    └── xxx_add_client_budgets.py ← DB-8
```

---

## DB Migrations

**DB-6 — Notifications**
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  org_id UUID REFERENCES organisations(id),
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  body TEXT,
  link TEXT,
  is_read BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX ON notifications(user_id, is_read);
```

**DB-7 — Audit Log**
```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_type TEXT NOT NULL,
  entity_id UUID NOT NULL,
  user_id UUID REFERENCES users(id),
  org_id UUID REFERENCES organisations(id),
  action TEXT NOT NULL,
  diff_json JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX ON audit_log(entity_type, entity_id);
```

**DB-8 — Client Budgets + Alerts**
```sql
CREATE TABLE client_budgets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organisations(id),
  client TEXT NOT NULL,
  month INTEGER NOT NULL,
  year INTEGER NOT NULL,
  budget_hours NUMERIC NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(org_id, client, month, year)
);
CREATE TABLE burn_rate_alerts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID REFERENCES organisations(id),
  client TEXT NOT NULL,
  threshold_pct INTEGER NOT NULL,
  hours_used NUMERIC,
  hours_budget NUMERIC,
  nova_summary TEXT,
  notified_at TIMESTAMP DEFAULT NOW()
);
```

---

## Week 4 Endpoints to Build

### Notifications (`backend/routes/notifications.py`)

| Endpoint | Method | Notes |
|---|---|---|
| /api/notifications | GET | Unread notifications for current user |
| /api/notifications/read-all | POST | Mark all as read |
| /api/notifications/:id/read | POST | Mark single as read |

Notification triggers to add (call from existing routes):
- Ticket assigned → notify new assignee
- @mention in comment → notify mentioned user
- Sprint started → notify all sprint members
- Standup generated → notify engineer

### Clients / Burn Rate (`backend/routes/clients.py`)

| Endpoint | Method | Notes |
|---|---|---|
| /api/clients/budget | POST | Set monthly hour budget per client (admin/manager) |
| /api/clients/burn-rate | GET | Current month burn % per client with NOVA summary |
| /api/clients/burn-rate/alerts | GET | History of fired alerts |

### Burn Rate Job (`backend/jobs/burnrate_job.py`)

```python
async def burnrate_job():
    # 1. For each org: get all client_budgets for current month
    # 2. Sum worklogs + manual_entries grouped by client
    # 3. Calculate burn % = hours_used / budget_hours * 100
    # 4. If burn % crosses 70/85/100/110 and alert not already sent:
    #    a. Generate NOVA summary: "Client X is at Y% with Z days remaining..."
    #    b. Insert into burn_rate_alerts
    #    c. Create notifications for PM + finance_viewer roles
```

---

## Week 4 — Performance + Polish Tasks

| Task | Notes |
|---|---|
| pgvector IVFFlat index | `CREATE INDEX ON ticket_embeddings USING ivfflat (embedding vector_cosine_ops) WITH (lists=100)` |
| pgvector IVFFlat index wiki | Same on wiki_embeddings |
| Health check endpoint | GET /api/health — returns DB status + NOVA status + version |
| Role permissions audit | Manually verify every endpoint: wrong role → 403 |
| CORS tighten | Only allow frontend origin in production config |
| Upload directory | Create `/uploads/` dir, serve static files via FastAPI |

---

## Week 4 Done Checklist

- [ ] Alembic migrations — notifications, audit_log, client_budgets, burn_rate_alerts
- [ ] `backend/routes/notifications.py` — list, mark read
- [ ] Notification triggers wired into ticket assign + @mention + sprint start
- [ ] `backend/routes/clients.py` — budget CRUD + burn rate endpoint
- [ ] `backend/jobs/burnrate_job.py` — hourly job, fires at 70/85/100/110%
- [ ] pgvector IVFFlat index on both embedding tables
- [ ] GET /api/health — returns all service statuses
- [ ] Role permissions audit — every endpoint tested with wrong role
- [ ] Test: set a client budget, log hours past 70%, alert fires in notifications

---

---

## Quick Reference — Start Each Session With

**Week 1:** "Read CLAUDE.md. I'm on Week 1. Build `backend/ai/nova.py` — the NOVA core engine. Use the code template in the Week 1 section."

**Week 2:** "Read CLAUDE.md. Week 1 is complete. I'm on Week 2. Start with the Alembic migration for wiki tables (DB-3), then build `backend/routes/wiki.py`."

**Week 3:** "Read CLAUDE.md. Weeks 1 and 2 are complete. I'm on Week 3. Start with the sprint DB migration, then `backend/routes/sprints.py`."

**Week 4:** "Read CLAUDE.md. Weeks 1–3 are complete. I'm on Week 4. Start with the notifications migration (DB-6), then `backend/routes/notifications.py`."

---

*Last updated: April 11, 2026*
*NOVA — Neural Orchestration & Velocity Assistant · Trackly · 3SC Hackathon 2026*

---
> Source: [anandverma0908/timesheet-tracker-backend](https://github.com/anandverma0908/timesheet-tracker-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
