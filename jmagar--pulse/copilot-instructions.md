## pulse

> *** YOU HAVE A MEMORY (neo4j-memory)-- SO USE IT****

# Claude Assistant Memory - Pulse Project

*** YOU HAVE A MEMORY (neo4j-memory)-- SO USE IT****
We save information from EVERY coding session to our memory, so take advantage of that information to give you context.

## Monorepo Structure

Multi-language monorepo combining Firecrawl API, MCP server, webhook bridge, file watcher, and web
UI.

**Node.js (pnpm workspace):**

- `apps/mcp` - Model Context Protocol server
- `apps/watcher` - Filesystem monitoring (indexes local files)
- `apps/web` - Next.js web UI (containerized with hot reload)
- `apps/claude` - Claude Code integration server
- `apps/firecrawl` - Firecrawl API (local build with PR #2381)
- `apps/changedetection` - changedetection.io configuration
- `packages/firecrawl-client` - Shared Firecrawl client
- `packages/logger` - Structured logging (Pino)

**Python (uv managed):**

- `apps/api` - FastAPI search bridge (hybrid vector/BM25)
- `apps/nuq-postgres` - PostgreSQL custom build

**Docker Deployment Architecture:**

**GPU Services** (deployed to gpu-machine via Docker context):
- TEI (embeddings) - Port 52000
- Qdrant (vector DB) - Ports 52001 (HTTP), 52002 (gRPC)
- Ollama (LLM) - Port 52003
- Deploy: `docker --context gpu-machine compose -f config/docker-compose.gpu-machine.yaml up -d`

**Local Services** (running on main host):
- Firecrawl API, Playwright, changedetection.io, Neo4j, Redis, PostgreSQL
- Deploy: `docker compose -f config/docker-compose.yaml up -d`

**Important**: When running from code-server container, services must be accessed via Tailscale IPs (not localhost):
- Local services: `<local-tailscale-ip>:<port>`
- GPU services: `<gpu-tailscale-ip>:<port>`
- Configure these IPs in your local `.env` file (not in tracked example files)

### Commands

```bash
# Stack control (use ./pulse from project root)
./pulse up [service|group]      # Start services
./pulse down [service|group]    # Stop services
./pulse restart [service|group] # Restart services
./pulse logs [service] [-n N]   # Follow logs (default: 100 lines)
./pulse ps [service|group]      # Show container status
./pulse health                  # Health check all services
./pulse exec <service> [cmd]    # Execute command in container

# Service groups: all, core, api, workers, web, monitoring, external
# Examples:
./pulse up core                 # Start postgres, redis, playwright
./pulse logs mcp                # Follow MCP logs
./pulse restart api             # Restart firecrawl, mcp, api

# Build
pnpm build                    # All (packages + apps)
pnpm build:{mcp,web,api}  # Individual

# Lint & Format
pnpm check                    # Check all (lint + format + typecheck)
pnpm lint                     # Lint (auto-fix)
pnpm format                   # Format (auto-fix)
pnpm ci:lint                  # Check only (no fix)

# Test
pnpm test                     # All
pnpm test:{mcp,web,api}   # Individual
./pulse test:up                # Start test containers (isolated)
./pulse test:down              # Stop test containers

# Dev
pnpm dev                      # All services parallel
pnpm dev:{mcp,web,api}    # Individual
```

## Service Ports & Naming

**Local Services:**

| Port     | Service         | Container              | Internal  | Notes                     |
| -------- | --------------- | ---------------------- | --------- | ------------------------- |
| 50100    | Playwright      | pulse_playwright       | 3000      | Browser automation        |
| 50102    | Firecrawl API   | firecrawl              | 3002      | Web scraping              |
| 50104    | Redis           | pulse_redis            | 6379      | Cache/queue               |
| 50105    | PostgreSQL      | pulse_postgres         | 5432      | Database                  |
| 50106    | File Watcher    | pulse_file-watcher     | N/A       | No HTTP                   |
| 50107    | MCP Server      | pulse_mcp              | 3060      | Claude integration        |
| 50108    | API             | pulse_api              | 52100     | Search API                |
| 50109    | changedetection | pulse_change-detection | 5000      | URL monitoring            |
| 50210-11 | Neo4j           | pulse_neo4j            | 7474/7687 | Graph database            |
| 50110    | Web UI          | pulse_web              | 3000      | Frontend                  |

**GPU Services** (deployed to gpu-machine):

| Port  | Service | Container    | Internal | Notes                 |
| ----- | ------- | ------------ | -------- | --------------------- |
| 52000 | TEI     | pulse_tei    | 3000     | Text embeddings       |
| 52001 | Qdrant  | pulse_qdrant | 6333     | Vector DB (HTTP)      |
| 52002 | Qdrant  | pulse_qdrant | 6334     | Vector DB (gRPC)      |
| 52003 | Ollama  | pulse_ollama | 11434    | LLM inference         |

**Container-to-Container URLs** (within Docker network `pulse`):

```
# Local services can reach each other via container names:
http://firecrawl:3002
http://pulse_mcp:3060
http://pulse_api:52100
redis://pulse_redis:6379
postgresql://pulse_postgres:5432/pulse_postgres

# To reach GPU services from local containers, use Tailscale IP (configure in .env):
http://<gpu-tailscale-ip>:52000  # TEI
http://<gpu-tailscale-ip>:52001  # Qdrant HTTP
http://<gpu-tailscale-ip>:52003  # Ollama
```

## Configuration System

**Hybrid architecture**: `config.yaml` (static) + `config/.env` (secrets/deployment)

### config.yaml (Static Configuration)

**Location**: `/mnt/cache/compose/pulse/config/config.yaml` (tracked in git)

**Purpose**: Static, non-secret configuration (ports, timeouts, feature flags, defaults)

**Mounted in Docker**: All services mount at `/etc/pulse/config.yaml:ro`

**Key sections**:

- `infrastructure`: Shared services, timezone, Docker paths
- `mcp`: Tool settings, OAuth, resource storage, HTTP client
- `webhook`: Database pools, search config, rate limits, cleanup
- `firecrawl`: Worker config, retry limits, browser settings
- `logging`: Default log levels per service

**Loading priority**:

1. Environment variable (highest priority)
2. `config.yaml` value
3. Application default (fallback)

### .env (Secrets & Deployment)

**Location**: `config/.env` (git-ignored, copy from `config/.env.example`)

**Purpose**: Secrets, credentials, deployment-specific values

**Key namespaces**:

**MCP (`MCP_*`):**

- Secrets: `MCP_FIRECRAWL_API_KEY`, `MCP_GOOGLE_CLIENT_SECRET`, `MCP_OAUTH_SESSION_SECRET`
- URLs: `MCP_FIRECRAWL_BASE_URL`, `MCP_WEBHOOK_BASE_URL`, `OPENAI_BASE_URL`
- Ports: `MCP_PORT` (external mapping)

**Services (`API_*` & `MCP_*`):**

- Secrets: `API_SECRET`
- URLs: `API_BASE_URL`
- Ports: `API_PORT` (external mapping)
- Database: `API_DATABASE_URL` (computed from `DATABASE_URL`)

**Shared:**

- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- `REDIS_URL`, `DATABASE_URL`
- `FIRECRAWL_API_KEY`, `CHANGEDETECTION_API_KEY`

### Adding New Configuration

**For static values** (ports, timeouts, defaults):

1. Add to `config.yaml` in appropriate section
2. Update TypeScript types in `packages/config/src/types.ts`
3. Update Python models in `apps/api/config/yaml_config.py`
4. Add tests for new config value
5. Update `config/.env.example` if env override is common

**For secrets/deployment values**:

1. Add to `config/.env` + `config/.env.example` with namespace prefix
2. Add to app config with AliasChoices/getEnvVar for backward compat
3. Update CLAUDE.md if service-wide
4. Document in `docs/configuration/migration-guide.md`

### Configuration References

- **Variable Implementation Guide**: `docs/HOW_TO_IMPLEMENT_VARIABLES.md`
- **Config Directory Context**: `config/CLAUDE.md`
- **Migration Guide**: `docs/configuration/migration-guide.md`
- **Implementation Plan**: `docs/plans/2025-12-04-env-config-migration.md`
- **config.yaml Schema**: `config/config.yaml` (config directory, v1.1.0)
- **.env Template**: `config/.env.example`

## Frontend Streaming Hooks (Abort Cleanup Pattern)

- Use abort-safe setters from `apps/web/hooks/utils/abort-safe-state.ts` to guard all `setState`
  calls (checks abort signal + mounted).
- Abort any in-flight stream before starting a new one; track `activeRequestId` to ignore late
  events and break loops on mismatched IDs.
- On abort or unmount, reset `streaming` and pending content, and null the controller inside
  `finally`.
- Tests: `apps/web/hooks/__tests__/use-chat.test.ts`,
  `apps/web/hooks/__tests__/use-project-chat.test.ts` must be updated with any behavior changes.


**Environment overrides in compose (only when needed):**

- Internal ports differing from external
- Service-specific URLs (container names)
- Dynamic values (`${VAR}` substitution)

**Volumes:** Bind mounts at `${APPDATA_BASE}/pulse_*` (default: `/mnt/cache/appdata/`)

## Dependency Injection (apps/api/deps/)

**Pattern:** Modular singleton providers organized into focused modules with generic factory
pattern.

**Structure:**

```
api/deps/
├── __init__.py       # Public API re-exports + backward compatibility shim
├── singleton.py      # Generic thread-safe Singleton factory with error caching
├── stubs.py          # Test mode stub implementations for all services
├── auth.py           # API authentication and webhook signature verification
├── infra.py          # Infrastructure providers (HTTP client, Redis, RQ queue)
├── services.py       # Service providers (embedding, search, indexing, etc.)
└── lifecycle.py      # Graceful shutdown with proper cleanup order
```

**Singleton Pattern:** All providers use a generic `Singleton[T]` class that handles:

- Thread-safe double-checked locking (async and sync)
- Error caching with TTL (prevents retry storms)
- Test mode stub injection via `settings.test_mode`
- Reset functionality for test isolation

**Usage:**

```python
# Modern imports (preferred)
from api.deps import get_http_client, verify_api_secret, cleanup_services

# Legacy test access (backward compatible via __getattr__ shim)
from api import deps
deps._text_chunker = None  # Reset singleton
```

**Providers by Category:**

| Category      | Module         | Functions                                                                                                                                                        |
| ------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auth**      | `auth.py`      | `verify_api_secret`, `verify_changedetection_signature`, `get_current_user`                                                                                      |
| **Infra**     | `infra.py`     | `get_http_client`, `get_redis_client`, `get_rq_queue`                                                                                                            |
| **Services**  | `services.py`  | `get_embedding_service`, `get_search_service`, `get_batch_service`, `get_indexing_service`, `get_metrics_service`, `get_webhook_service`, `get_api_info_service` |
| **Lifecycle** | `lifecycle.py` | `cleanup_services`, `graceful_shutdown`                                                                                                                          |

**Backward Compatibility:** The `__init__.py` provides a `__getattr__`/`__setattr__` shim that
allows legacy test code to access singleton instances via `deps._singleton_name`. All existing
imports work unchanged.

## Key Integration Points

**MCP ↔ Firecrawl:** Scraping (scrape, map, crawl, profile_crawl tools) **MCP ↔ Webhook:** Search
(query tool), metrics (profile_crawl) **changedetection ↔ Webhook:** Change notifications
**PostgreSQL schemas:** `public` (Firecrawl), `webhook` (metrics/search)

**Firecrawl:** Local build with PR #2381 (prevents retry loops, handles disconnects)

**BM25 Eviction:** BM25Engine now uses LRU eviction with access tracking during search and document
addition; falls back to FIFO when access history is unavailable. See
`docs/configuration/bm25-lru-eviction.md` for behavior and configuration notes.

**File Descriptor Management:** Watcher monitors FD usage (Linux) and throttles when approaching
ulimit (default: 1024). See `apps/watcher/src/fd-monitor.ts` for implementation.

## Type Safety (Python API)

**TypedDict Migration Completed:** All service layer returns use TypedDict for compile-time type checking.

**Type Definitions:** Located in `apps/api/type_defs/`
- `search.py` - Search result types (VectorSearchResult, BM25SearchResult, HybridSearchResult)
- `indexing.py` - Indexing result types (ChunkResult, IndexingResult, BatchIndexingResult)
- `vector_store.py` - Vector operation types (VectorSearchResponse, VectorCountResponse, VectorDeleteResponse)
- `bm25.py` - BM25 result types (BM25SearchResponse, BM25AddResponse, BM25StatsResponse)

**Benefits:** Full IDE autocomplete, compile-time validation with mypy --strict, self-documenting code.

**Details:** See `apps/api/docs/TYPE_SAFETY.md` for comprehensive documentation, examples, and migration status.

## Testing

**CRITICAL:** Tests use isolated containers (port +5000 offset)

| Service    | Production Port | Test Port | Notes                                  |
| ---------- | --------------- | --------- | -------------------------------------- |
| PostgreSQL | 50105           | 55105     | Isolated test database                 |
| Redis      | 50104           | 55104     | Isolated test cache                    |
| Qdrant     | 52001 (GPU)     | 55001     | Isolated test vector DB (on local)     |

**Test setup:**

```bash
docker compose -f config/docker-compose.test.yaml up -d
cp config/.env.test.example config/.env.test  # Configure test env
pnpm test
```

**Test env configuration:**

```bash
# Configure in .env.test with appropriate host IPs
# Use localhost for direct host access, or Tailscale IP from code-server
API_DATABASE_URL=postgresql+asyncpg://test_user:test_password@<host>:55105/test_db
API_REDIS_URL=redis://<host>:55104
API_QDRANT_URL=http://<host>:55001
API_SKIP_DB_FIXTURES=0
```

**Important**:
- Test containers run on local machine (NOT gpu-machine)
- Configure appropriate host IP in your local `.env.test`
- Never use production ports/containers in tests!

## Logging

**Structured logging:** TypeScript (Pino), Python (structlog)

**Core fields:** `service`, `level`, `timestamp`, `message`, `correlation_id`, `context`

**TypeScript:**

```typescript
import { createLogger, withContext } from "@pulse/logger";
const logger = createLogger("pulse_mcp");
const toolLogger = withContext(logger, "scrape_tool");
```

**Python:**

```python
from utils.logging import get_service_logger
logger = get_service_logger(context="indexing")
```

**Env vars:** `LOG_LEVEL`, `LOG_FORMAT` (TS); `API_LOG_LEVEL`, `API_ENVIRONMENT` (Python)

## Test-Driven Development (TDD)

**MANDATORY:** RED (failing test) → GREEN (minimal code) → REFACTOR

**Standards:**

- Unit tests (mocked deps), integration (real services), E2E (Playwright)
- Isolated, deterministic, fast, 85%+ coverage
- Descriptive names: `test_create_user_rejects_invalid_email`

**Stack:** pytest + pytest-asyncio (Python), Jest + RTL (TypeScript)

## Implementing Plans

When creating a bead for a plan, include in the description:

```
**Implementation Plan:** See `docs/plans/YYYY-MM-DD-feature-name.md`

**To implement:** Use `/superpowers:subagent-driven-development docs/plans/YYYY-MM-DD-feature-name.md` to execute the plan.
```

This enables subagent-driven execution of the plan tasks.

See README.md and QUICKSTART.md for details.

## Error Handling

**Documentation:** `docs/development/error-handling-best-practices.md`

**Python pattern:**

```python
logger.error("Message", error=str(e), error_type=type(e).__name__, exc_info=True)
```

**TypeScript pattern:**

```typescript
import { getErrorDetails } from "@pulse/errors";
const details = getErrorDetails(error);
logger.error({ error: details.message, errorType: details.type, stack: details.stack }, "Message");
```

**Key utilities:**

- `@pulse/errors` - TypeScript error normalization (`normalizeError`, `getErrorDetails`)
- `utils/error_context.py` - Python error enrichment (`enrich_error_context`, `ErrorContext`)
- `utils/error_sanitizer.py` - Sanitize errors for external responses (`sanitize_error`, `get_safe_error_response`)

**Rules:**

1. Never swallow errors silently (empty catch blocks)
2. Always include `error_type` in error logs
3. Use `exc_info=True` for critical paths
4. Sanitize errors before exposing to API clients

## Shared Schemas (@pulse/schemas)

**Source of truth:** TypeScript Zod schemas in `packages/schemas/src/`

**Generation pipeline:** Zod → JSON Schema → Pydantic

**Covered domains:**

- Browser actions (click, wait, scroll, etc.)
- Search (request/response, filters, modes)
- Scrape (request/response, parsers, result handling)
- Metrics (profile request, crawl metrics, timing)

**Regenerate after schema changes:**

```bash
pnpm --filter @pulse/schemas generate
cp packages/schemas/generated/python/*.py apps/api/schemas/generated/
```

**CI:** `.github/workflows/schemas.yaml` verifies sync on PR

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmagar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jmagar)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
