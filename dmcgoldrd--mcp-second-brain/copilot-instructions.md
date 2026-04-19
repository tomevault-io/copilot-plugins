## mcp-second-brain

> Personal AI Memory — works with Claude, ChatGPT, Gemini, Cursor, and any MCP client.

# MCP Brain

Personal AI Memory — works with Claude, ChatGPT, Gemini, Cursor, and any MCP client.

## Stack

| Component | Technology |
|-----------|-----------|
| MCP Server | Python 3.12 + FastMCP 2.x + uvicorn |
| Database | Supabase Postgres + pgvector (HNSW, 1536 dims) |
| Auth | Supabase Auth (OAuth 2.1, ES256 JWTs) |
| Embeddings | OpenAI text-embedding-3-small |
| Search | Hybrid: pgvector cosine + Postgres FTS + RRF ranking |
| Billing | Stripe ($7.99/month) |
| Package Manager | uv (Python), bun (frontend) |
| Deployment | Railway (Docker, multi-stage build) |

## Architecture

```
MCP Client → POST /mcp/ (Streamable HTTP) → FastMCP Server → Supabase Postgres
                                                ↓
                                         OpenAI Embeddings
```

- Auth: Supabase JWT validated via JWKS. User ID from `sub` claim.
- Banks: Isolated memory namespaces per user. Selected via `x-bank-slug` header.
- RLS: Row-level security on all tables. Users only see their own data.

## Key Files

```
src/
├── main.py           # Entry point, serves MCP + frontend
├── server.py         # FastMCP server, 7 MCP tools, _resolve_auth()
├── config.py         # Environment variables
├── embeddings.py     # OpenAI embedding generation
├── metadata.py       # Heuristic metadata extraction + type classification
├── ratelimit.py      # Token bucket rate limiter (in-memory)
├── db/
│   ├── connection.py # asyncpg pool with pgvector
│   ├── memories.py   # Memory CRUD (create, search, list, delete, stats)
│   ├── banks.py      # Bank CRUD
│   └── profiles.py   # Profile/subscription queries
└── tools/
    └── memory_tools.py  # Business logic layer
```

## MCP Tools

| Tool | Description |
|------|-------------|
| `create_memory` | Store memory with auto-embedding, classification, metadata |
| `search_memories` | Hybrid semantic + full-text search with RRF scoring |
| `list_memories` | Browse recent memories, filter by type |
| `delete_memory` | Remove a specific memory |
| `brain_stats` | Memory count, type breakdown, date range |
| `list_banks` | List memory banks for user |
| `create_bank` | Create new memory bank |

## Database Schema

- `profiles`: user_id, stripe_customer_id, subscription_status, memory_count
- `memories`: content, embedding (vector 1536), metadata (JSONB), memory_type, tags, source, fts (tsvector)
- `banks`: user_id, name, slug, is_default
- `subscriptions`: stripe data

## Commands

```bash
uv sync --all-extras          # Install all dependencies
uv run python -m src.main     # Start server
uv run pytest                 # Run tests (148 tests, 97% coverage)
uv run ruff check src/ tests/ # Lint
uv run ruff format src/ tests/ # Format
```

## Memory Types

observation, task, idea, reference, person_note, decision, preference

## Coding Conventions

- All DB queries use asyncpg with parameterized queries (no SQL injection)
- All user inputs validated in server.py before reaching business logic
- Atomic limit checks use `FOR UPDATE` row locks to prevent TOCTOU races
- Rate limiting via in-memory token bucket (tool_limiter: 60/min, embedding_limiter: 30/min)
- RLS policies enforce per-user data isolation at DB level
- Errors returned as JSON dicts with `status` and `error` fields, never raised as exceptions to MCP clients

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmcgoldrd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
