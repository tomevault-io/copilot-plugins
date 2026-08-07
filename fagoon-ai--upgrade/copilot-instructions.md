## upgrade

> This repository contains the backend for the Fagoon AI Agents Workflow platform, built with FastAPI. It handles complex AI orchestration, including RAG-enabled chat, background video generation, image generation, text-to-speech, tool-calling execution, and deep integration with Google Workspace.

# Fagoon AI Agents Workflow

This repository contains the backend for the Fagoon AI Agents Workflow platform, built with FastAPI. It handles complex AI orchestration, including RAG-enabled chat, background video generation, image generation, text-to-speech, tool-calling execution, and deep integration with Google Workspace.

---

## Mandatory Rule for AI Agents
**CRITICAL:** Any major architectural, structural, or infrastructural changes made to this project MUST be documented by updating this `GEMINI.md` file immediately. This ensures all future agents understand the current system state. Do not proceed with major architecture refactors without updating this document.

---

## Tech Stack & Architecture

- **Framework:** FastAPI (Python 3.12+)
- **Package Management:** `uv` with `pyproject.toml`
- **Primary Database:** PostgreSQL (accessed via SQLAlchemy `AsyncSession` & `asyncpg`)
- **Vector Database:** PostgreSQL via `pgvector` (for RAG document chunks)
- **Migrations:** Alembic
- **Background Jobs / Queues:** Celery (with Redis broker, Eventlet execution pool) or Inline Async Queue (Lite Mode).
- **Real-time Communication:** Server-Sent Events (SSE) for streaming LLM responses, WebSockets for video generation status.
- **LLM/AI Integrations:** OpenAI, Anthropic, Google GenAI (Gemini/Veo), Groq, Hugging Face, Fal AI, ElevenLabs, Kokoro.
- **Architecture Pattern:** Layered/Modular (Routers -> Services -> Providers -> Data/Storage).

---

## Dual-Mode Runtime Architecture (Lite & Full)

Fagoon supports two modes of operation:
1. **Lite Mode** (Default/Single-user local package):
   - **Launch Command:** `fagoon up` or with local fallbacks (`fagoon up --ollama`).
   - **Infrastructure:** Runs entirely self-contained without heavy external dependencies. It uses:
     - **In-memory Limiter:** `MemoryRateLimiter`
     - **In-memory Cache:** `MemoryCache`
     - **Inline Async Task Queue:** `InlineTaskQueueWithSyncSupport`
     - Single-process concurrency model (`WEB_CONCURRENCY=1` enforced at startup).
     - Secrets (such as `JWT_SECRET` and `encryption_key`) are auto-generated on first boot and persisted to `<DATA_DIR>/config.json`.
   - **No Redis/Celery imports:** Business logic modules must never import `redis` or `celery` directly. They route all requests through `app.state.limiter`, `app.state.queue`, and `app.state.cache`. This restriction is validated by `import-linter`.
2. **Full Mode** (Production multi-user environment):
   - **Launch Command:** `docker compose -f deploy/docker-compose.full.yml up --build` or setting environment variable `LITE_MODE=false`.
   - **Infrastructure:** Employs Redis as a broker and Celery with an Eventlet execution pool for distributed, multi-worker asynchronous background processing (such as long-running video generation).

Mode selection is governed by the `LITE_MODE` environment variable, resolved at startup in `src/core/runtime.py`.

---

## Strict Layer & Module Rules

- **Routers (`src/api/v1/routers/`):** ONLY routing and request validation (Pydantic). Dependency injection happens here. No business logic. Subdirectories categorize domains (`agents`, `authentication`, `external`, `misc`, `upgrade`, `video_gen`, `workflow`, `workspace`).
- **Services (`src/services/`):** Core business logic. Orchestrates calls to models, storage, and external providers.
    - `tool_handlers/`: Specific execution logic for AI tools (e.g., `web_search_handler.py`, `mermaid_handler.py`, `deep_research_handler.py`).
    - `rag/`: Document ingestion and chunking (`ingestion_manager.py`).
    - `google_workspace/`: Logic for Docs, Drive, and Gmail APIs.
    - `channel_adapter/`: Webhook processing for external messaging platforms.
    - `nosql/`: Database wrapper services (Legacy naming, but currently houses `postgres_services.py`).
- **Models & Schemas:** 
    - `src/models/sql/`: SQLAlchemy Base models (`models.py`) defining the exact PostgreSQL schema.
    - `src/schemas/`: Pydantic schemas for request/response validation. Keep them strictly typed.
- **Storages (`src/storages/`):** Abstractions for `file_storage` and vector storage (`vectordb_storages/pgvector.py`).
- **LLMs & Providers (`src/llms/`, `src/providers/`):** Wrappers for external AI API clients.
- **Core (`src/core/`):** Application globals, settings (`settings.py`), database connection pool (`PostgresManager`), exception handlers.

---

## Lifespan Singletons
Heavy clients are initialized once during FastAPI lifespan (`src/launch_server.py`) and attached to `app.state`. Do NOT instantiate these per request:
- `app.state.postgres_manager`: Connection pool for PostgreSQL.
- `app.state.httpx_client`: Global HTTP client.
- `app.state.vector_store`: Singleton `PgVectorStorage`.
- `app.state.agent_manager` & `app.state.chat_orchestrator`: Core orchestration singletons.

---

## Core Request Lifecycles & Component Flows

### 1. RAG-Enabled Agent Chat (Server-Sent Events)
The system uses Retrieval-Augmented Generation (RAG) to provide AI agents with specific document context.
1. **Entry Point:** `POST /api/v1/agent/chat` (via `src/api/v1/routers/agents/agent_chat.py`).
2. **Middleware Check:** `AuthMiddleware` (`src/api/custom_middleware.py`) validates the JWT in headers/cookies.
3. **Validation & Routing:** Validates `AgentChatRequest`. Retrieves `ChatOrchestrator` from `app.state`.
4. **Orchestrator Execution (`src/services/agents/chat_orchestrator.py`):**
    - **Embed:** Calls the embedding model to generate a vector for the user query.
    - **Retrieve:** Queries PostgreSQL (`app.state.vector_store`) asynchronously using `pgvector` to retrieve relevant `document_chunks`. Offloads BM25 Reranking to a background thread if configured.
    - **Tool Handling:** If the LLM requests a tool, execution routes to `src/services/tool_handlers/` (e.g., triggering a deep web search or generating Mermaid charts).
5. **Streaming Response:** Yields Server-Sent Events (SSE) back to the client using utility functions in `src/utils/common.py`.
6. **Persistence:** Saves chat history to PostgreSQL asynchronously via `AgentChatService`.

### 2. Background Video Generation
1. **Entry Point:** `/api/v1/video-generation/...` (via `src/api/v1/routers/video_gen/video_gen_routes.py`).
2. **Database Initiation:** A `VideoJob` record is created in PostgreSQL with status `PENDING`.
3. **Task Queueing:** A Celery task (`generate_video_task`) is dispatched to the broker (Redis) in Full mode, or ran inline as an asyncio task in Lite mode. The HTTP request immediately returns the `job_id` to the client.
4. **Worker Execution:**
    - Worker/Inline runner updates PostgreSQL status to `PROCESSING`.
    - Dispatches to `VeoVideoGenerator` (`src/services/veo_video_generator.py`) or text-to-video handlers.
    - Resulting video is saved locally or pushed to Google Cloud Storage (GCS).
    - Status is updated to `COMPLETED` or `FAILED`.
5. **Client Notification:** A background asyncio task (`periodic_status_broadcaster` in `src/video_generation/websocket_manager.py`) constantly reads job statuses from the DB and pushes real-time updates to the client via WebSockets.

### 3. Google Workspace Operations
1. **Authentication:** User authenticates via `/api/v1/google-auth/login` (`src/api/v1/routers/workspace/google_auth.py`). System exchanges code for access/refresh tokens and saves them in PostgreSQL (`google_tokens` table).
2. **Operation:** Request sent to a workspace endpoint (e.g., `/api/v1/gmail/messages` in `src/api/v1/routers/workspace/gmail.py`).
3. **Dependency Check:** Router depends on `get_google_credentials` (`src/api/v1/routers/workspace/deps.py`), verifying the Google token exists and is valid.
4. **Service Call:** Service classes in `src/services/google_workspace/` use the `google-api-python-client` to execute the operation on behalf of the user.

### 4. Authentication Flow
1. **Middleware Check:** `AuthMiddleware` runs on all non-public routes (bypassing paths in `ALLOWED_URL_PATH_WITHOUT_AUTHORIZATION`).
2. **Token Extraction & Validation:** Attempts to read `Authorization` header, falls back to `jwt` cookie. Decodes JWT using `JWT_SECRET`. Verifies user active status.
3. **Auto-Refresh:** If the access token is expired but a valid `refresh_token` cookie exists, generates a new access/refresh pair, updates context, and appends cookies to the response.
4. **Authorization Injection:** Injects `request.state.user` for specific router dependencies. Routes are managed in `src/api/v1/routers/authentication/auth_router.py`.

---

## Database Switching & Live Migration

Users can transition the platform database from the bundled local PostgreSQL instance to an external, user-supplied PostgreSQL URL (e.g., self-hosted or cloud database).

### 1. The Switch Service (`src/services/database/switch_service.py`)
The migration process executes sequentially to guarantee zero-downtime safety (the local DB is left intact as a fallback):
1. **Validate:** Establishes connection to target with the supplied user roles.
2. **Pgvector Preflight:** Proactively verifies that the `pgvector` extension is available and can be enabled on the target. If missing, it fails fast with a clear error message.
3. **Preflight Privileges:** Confirms that the database role has schema creation/table dropping permissions.
4. **Schema Migration:** Performs an `alembic upgrade head` against the target database.
5. **Data Copy (Optional):** Employs `pg_dump` (data only) from local and restores it into the target via `pg_restore`.
6. **Persist:** Updates the new database URL in the configuration file (`<DATA_DIR>/config.json`).
7. **Restart:** Sets a restart flag. Since database pools cannot hot-swap dynamically, the server restarts to initialize the new PostgreSQL pool.

### 2. Status Polling
All migration phases and progress states are logged to a file-based status handler (`<DATA_DIR>/db_switch_status.json`). Polling is file-based to remain active even when the database connection is interrupted.

---

## Required Environment Variables (from `settings.py`)

- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_HOST`, `POSTGRES_PORT` (for database connection URL)
- `JWT_SECRET`, `JWT_ALGORITHM`
- `CELERY_BROKER_URL`, `CELERY_RESULT_BACKEND`
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- `OPENAI_API_KEY`, `GROQ_API_KEY`, `GEMINI_API_KEY`, etc. (for respective features)

---

## Development, Quality, and Testing Workflow

### 1. Running Tests
Tests are executed via `pytest`. To bypass collection errors in auxiliary scripts outside the core test suite, target the `tests` folder directly:
```bash
# Run all core tests
uv run pytest tests
```

### 2. Guarding Architecture Boundaries
We enforce a strict physical separation of dependencies. Direct import of `redis` or `celery` is forbidden in general business/API modules. This is validated via:
- **Import Linter:** Configured via `.importlinter`:
  ```bash
  uv run lint-imports
  ```
- **Import Guard Script:** Asserts string-level cases:
  ```bash
  ./scripts/guard_imports.sh
  ```

### 3. Code Quality (Linting & Formatting)
We enforce clean code styling using `ruff`:
```bash
uv run ruff check .
```

### 4. Database Migrations
Always apply pending migrations using Alembic helper or manual commands:
```bash
# Recommended PowerShell helper
.\alembic_helper.ps1 upgrade head

# Manual commands
alembic upgrade head
```

---

## What NOT to do (Anti-patterns)

- **Do NOT** import `redis` or `celery` directly in API routers or general service modules. Use `app.state.limiter`, `app.state.queue`, or `app.state.cache` instead.
- **Do NOT** put core business logic inside FastAPI router functions.
- **Do NOT** use synchronous code for database operations or remote network calls. Always use async variants or execute blocking sync code inside thread executors.
- **Do NOT** instantiate heavy clients (such as DB pools, HTTP clients, or AI model registries) per request. Always register and fetch them as lifespan singletons on `app.state`.
- **Do NOT** bypass the custom `AuthMiddleware` for protected routes. Avoid adding unprotected pathways unless specifically approved.
- **Do NOT** suppress or bypass Pydantic validation errors. Maintain strict schemas.
- **Do NOT** attempt to hot-swap database pools without a server restart. Establish restart mechanisms instead.

---
> Source: [Fagoon-AI/upgrade](https://github.com/Fagoon-AI/upgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
