## medchatbot-1

> FastAPI backend for a Malaysian medical chatbot.

# CLAUDE.md

## What This Is
FastAPI backend for a Malaysian medical chatbot.
One feature inside an Android app. Group FYP project.
Deployed on Azure App Service. Uses PostgreSQL for persistence and Redis for caching.

## What Is Already Done (DO NOT REBUILD)
A Colab notebook already:
- Scraped MOH CPG PDFs + FUKKM drug table
- Chunked, embedded, and uploaded everything to Pinecone
- Built bm25_index.pkl (in this folder — loaded at startup)

Backend only READS from Pinecone. Never writes. Never embeds. Never upserts.

## Two Answer Modes
- Score >= 0.70 → Grounded mode: answer uses CPG context + cite sources
- Score <  0.70 → Knowledge mode: MedGemma uses own knowledge + add disclaimer

Disclaimer (English): "Note: This answer is based on general medical knowledge. Please consult your doctor."
Disclaimer (Malay): "Nota: Jawapan ini berdasarkan pengetahuan perubatan umum. Sila rujuk doktor anda."

## Pinecone Data — DO NOT GUESS THESE FIELDS

Children namespace = "" (searchable vectors, 768 dims):
```
metadata keys: parent_id, title, source, type, text (max 800 chars), word_count, chunk_num, chunk_type ("child")
```

Parents namespace = "parents" (zero vectors, fetched by ID only):
```
metadata keys: title, source, type, text (max 3000 chars), word_count, chunk_type ("parent"), chunk_num
```

Never query parents by vector — fetch by ID only (they have zero vectors).

## Database — PostgreSQL
Used for three things:
1. **parent_chunks table** — load parents.json once into PostgreSQL. At runtime, fetch parent text from DB instead of from file or Pinecone.
2. **chat_history table** — persist conversations per session_id so mobile app can restore history.
3. **qa_logs table** — log every question + answer + metadata for analytics.

PostgreSQL is the source of truth for parent chunk text. Pinecone parent fetch is no longer used as primary — PostgreSQL replaces it. Keep parents.json as a one-time import seed only.

## Cache — Redis
Used for three things:
1. **Answer cache** — cache full ChatResponse JSON keyed by normalized query. TTL = 1 hour.
2. **Translation cache** — cache Malay→English translations to skip Gemini calls. TTL = 24 hours.
3. **Session store** — store session metadata (e.g., language preference). TTL = 30 minutes.

Cache key patterns:
- `answer:{sha256(query)}` → cached ChatResponse JSON
- `translate:{sha256(malay_query)}` → English translation string
- `session:{session_id}` → session JSON

## Key Rules
1. Load ALL ML models once at startup using FastAPI lifespan, store on app.state
2. Never load models inside request handlers
3. BM25 tokenizer must match Colab exactly (see MODELS.md)
4. Pinecone children namespace = "" / parents namespace = "parents"
5. Never query parents by vector — fetch by ID only
6. MedGemma runs on external GPU VM — call via HTTP POST. **Not built yet — if MEDGEMMA_URL is empty, skip and use Gemini Flash directly.**
7. Always fallback to Gemini Flash if MedGemma is unreachable
8. Never hardcode API keys or connection strings — use .env
9. CORS allow all origins
10. Chat history is loaded from PostgreSQL by session_id — NOT sent in request body
11. PostgreSQL connections via async pool (asyncpg)
12. Redis connections via async client (redis.asyncio)
13. On startup: check DB tables exist, run migrations if needed

## Environment Variables (.env)
```
# Pinecone
PINECONE_API_KEY=...
PINECONE_INDEX=...

# Google
GEMINI_API_KEY=...

# MedGemma (leave empty until GPU VM is deployed)
MEDGEMMA_URL=

# PostgreSQL (Azure Database for PostgreSQL)
DATABASE_URL=postgresql+asyncpg://user:pass@host:5432/medbot

# Redis (Azure Cache for Redis)
REDIS_URL=rediss://default:password@host:6380/0
```

## Project Structure
```
backend/
├── .env
├── requirements.txt
├── Dockerfile
├── bm25_index.pkl
├── parents.json              ← seed file only, imported to DB once
├── alembic.ini               ← DB migration config
├── alembic/
│   └── versions/
└── app/
    ├── main.py
    ├── config.py
    ├── database.py            ← async engine, session factory, Redis client
    ├── models/
    │   ├── schemas.py         ← Pydantic request/response models
    │   └── db_models.py       ← SQLAlchemy ORM models
    ├── retrieval/
    │   ├── embedder.py
    │   ├── query_processor.py
    │   ├── hybrid_search.py
    │   ├── reranker.py
    │   └── retriever.py
    ├── generation/
    │   └── llm.py
    ├── services/
    │   ├── cache_service.py   ← Redis get/set helpers
    │   ├── chat_service.py    ← save/load chat history
    │   └── log_service.py     ← log Q&A to qa_logs
    └── scripts/
        └── seed_parents.py    ← one-time: load parents.json → PostgreSQL
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BoXuan0630)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BoXuan0630)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
