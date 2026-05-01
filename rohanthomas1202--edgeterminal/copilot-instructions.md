## edgeterminal

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**EdgeTerminalUI**
You are a senior UI designer and frontend developer.
Build premium, dark-themed interfaces.
Use subtle animations, proper spacing, and visual hierarchy.
No emoji icons. No inline styles. No generic gradients.

**EdgeTerminal**

A real-time cross-market arbitrage monitoring terminal for retail prediction market traders. The system continuously ingests data from Polymarket and Kalshi, matches equivalent contracts across venues, and surfaces price discrepancies as actionable arbitrage opportunities in a live dashboard. The core engine is Polymarket-Kalshi matching and pricing; the terminal wraps this in a broader trading dashboard experience.

**Core Value:** When two prediction markets disagree on the same event, show the trader exactly where, by how much, and with what confidence — in real time, with historical proof that detected opportunities were real.

### Constraints

- **Tech stack**: Python 3.12 + FastAPI backend, Next.js + React frontend, PostgreSQL — already established
- **Free APIs only**: No paid data feeds; must work within Polymarket and Kalshi public API rate limits
- **Regulatory**: Frame as research/analysis tool, not financial advice; no automated execution without explicit user action
- **API rate limits**: Both venues rate-limit; polling strategy must be sustainable at 30-second intervals
- **Team coordination**: 15-20 people requires clear module boundaries and well-defined interfaces
- **Matching accuracy**: False arb signals from bad matching are worse than missing real arbs — precision over recall
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Python 3.12 - Backend API, market ingestion, matching, and scheduling
- TypeScript 5 - Frontend React components, type-safe API client, utilities
- JavaScript - Build configuration (ESLint, PostCSS, Next.js config)
- SQL - PostgreSQL database schema and queries (via SQLAlchemy ORM)
## Runtime
- Python 3.12.x (Docker: `python:3.12-slim`)
- Node.js 20.x (Docker: `node:20-alpine`)
- pip - Python dependency management
- npm - JavaScript/Node.js dependency management (lockfile: `package-lock.json`)
## Frameworks
- FastAPI 0.115.0 - RESTful API, dependency injection, async handlers
- Uvicorn 0.32.0 - ASGI server (FastAPI host)
- SQLAlchemy 2.0.49 (with asyncio) - Async ORM, database models, relationships
- APScheduler 3.11.0 - Background job scheduling, periodic polling and matching tasks
- Next.js 16.2.3 - React meta-framework, SSR, routing, build tooling
- React 19.2.4 - UI component framework
- React DOM 19.2.4 - DOM rendering for React
- pytest - Python test runner and assertion library
## Key Dependencies
- sqlalchemy[asyncio] 2.0.49 - Async ORM with PostgreSQL support
- asyncpg 0.30.0 - PostgreSQL async driver
- alembic 1.14.0 - Database migration tool (schema versioning)
- httpx 0.28.0 - Async HTTP client (for Kalshi/Polymarket API calls)
- anthropic 0.43.0 - Claude API client for LLM verification of market matches
- sentence-transformers 5.3.0 - Semantic embedding model (all-MiniLM-L6-v2, ~90MB) for embedding-based matching
- torch 2.11.0 - PyTorch deep learning framework (dependency of sentence-transformers)
- numpy 2.4.4 - Numerical computing (used by embeddings)
- pydantic-settings 2.7.0 - Configuration management from environment variables
- python-dotenv 1.0.1 - Load .env files for local development
- cryptography 44.0.0 - RSA signing for Kalshi API authentication
- swr 2.4.1 - Stale-while-revalidate data fetching hook, auto-refetch with 30s intervals
- tailwindcss 4 - Utility-first CSS framework
- @tailwindcss/postcss 4 - PostCSS plugin for Tailwind
- tailwind-merge 3.5.0 - Merge and deduplicate Tailwind classes
- tw-animate-css 1.4.0 - Tailwind animation utilities
- lightweight-charts 5.1.0 - TradingView-style price charts (SpreadChart)
- clsx 2.1.1 - Conditional className utility
## Configuration
- `DATABASE_URL` - PostgreSQL connection string (default: `postgresql+asyncpg://truthlayer:truthlayer@localhost:5432/truthlayer`)
- `KALSHI_KEY_ID` - Kalshi API key identifier
- `KALSHI_PRIVATE_KEY_PATH` - Path to RSA private key for Kalshi signing
- `ANTHROPIC_API_KEY` - Claude API key for LLM verification
- `POLLING_INTERVAL_SECONDS` - How often to fetch market data (default: 30)
- `MATCHING_INTERVAL_MINUTES` - How often to run matching algorithm (default: 15)
- `LOG_LEVEL` - Logging verbosity (default: INFO)
- `NEXT_PUBLIC_API_URL` - Backend API endpoint (e.g., `http://localhost:8000`)
- `.prettierrc` - Not present; using ESLint defaults
- `eslint.config.mjs` - ESLint 9 flat config with Next.js core web vitals and TypeScript rules
- `next.config.ts` - Minimal Next.js configuration
- `tsconfig.json` - TypeScript compiler options (target ES2017, strict mode, `@/*` path alias)
- `postcss.config.mjs` - PostCSS configuration with Tailwind v4
- `pyproject.toml` - Not present (using pip + requirements.txt)
## Platform Requirements
- Docker & Docker Compose for containerized local development
- PostgreSQL 16 (provided via docker-compose)
- Python 3.12+
- Node.js 20+
- Docker images for backend and frontend
- PostgreSQL 16+ database
- Uvicorn ASGI server (or production ASGI server like Gunicorn + Uvicorn workers)
- Node.js runtime only needed for Next.js build step; frontend serves static/SSR from Vercel, Netlify, or similar
## Database
- Async driver: asyncpg
- ORM: SQLAlchemy with asyncio support
- Migrations: Alembic (schema versioning)
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- TypeScript/React: `PascalCase` for components (e.g., `NavBar.tsx`, `EventRow.tsx`), `camelCase` for utilities and hooks (e.g., `api.ts`, `types.ts`)
- Python: `snake_case` for modules and files (e.g., `polymarket.py`, `scheduler.py`), class names in `PascalCase` (e.g., `PolymarketClient`, `KalshiMarket`)
- TypeScript: `camelCase` for all functions and hooks (e.g., `useEvents()`, `handleSearch()`, `formatVolume()`)
- Python: `snake_case` for all functions (e.g., `compute_similarity()`, `parse_market()`, `_auth_headers()`)
- Private/internal functions: Prefixed with underscore in Python (e.g., `_tokenize()`, `_stem()`, `_load_private_key()`)
- TypeScript: `camelCase` (e.g., `expandedId`, `searchValue`, `isLoading`)
- Python: `snake_case` (e.g., `poly_markets`, `last_matching_run`, `timestamp_ms`)
- Constants: ALL_CAPS_SNAKE_CASE (e.g., `AUTO_MATCH_THRESHOLD`, `RELEVANT_CATEGORIES`, `KALSHI_BASE`)
- TypeScript interfaces: `PascalCase` with `Props` suffix for component props (e.g., `FilterBarProps`, `EventRowProps`)
- Python dataclasses: `PascalCase` (e.g., `PolymarketMarket`, `KalshiMarket`, `MatchResult`)
- Pydantic models: `PascalCase` (e.g., `EventSummary`, `EventDetail`)
## Code Style
- ESLint + Next.js defaults (flat config format in `eslint.config.mjs`)
- No Prettier config; ESLint handles linting only
- TypeScript strict mode enabled (`"strict": true` in tsconfig.json)
- Line wrapping: Follows ESLint defaults; natural breaks around 100 characters
- ESLint config: `frontend/eslint.config.mjs` with Next.js Web Vitals and TypeScript presets
- No Python linting config detected; follows PEP 8 conventions by convention
- No formatting tool enforces Python style
- TypeScript/TSX: 2 spaces (Next.js default)
- Python: 4 spaces (PEP 8 standard)
## Import Organization
- TypeScript: `@/*` → `src/*` (defined in `tsconfig.json`)
- Python: Direct module imports from package root (e.g., `from app.config import settings`)
## Error Handling
- Components use optional chaining and nullish coalescing (`?.`, `??`)
- API fetcher throws on non-200 status: `if (!res.ok) throw new Error(...)`
- Loading states managed via `isLoading` flag from SWR
- No try-catch in components; rely on parent error boundaries
- Extensive try-catch blocks in service functions
- Logging errors with context via `logger.error()` or `logger.warning()`
- Graceful degradation: catch exceptions, log, continue processing
- Database operations wrapped in exception handlers to avoid rollback cascades
## Logging
- Logger created per module: `logger = logging.getLogger(__name__)`
- Log level configured via env var: `settings.log_level` (defaults to "INFO")
- Log messages are descriptive and include relevant context (IDs, counts, errors)
## Comments
- Private function docstrings: Explain parameters, return values, and algorithm logic
- Complex business logic: Document the "why" behind thresholds and calculations
- API contract explanations: Describe query parameters and response structure
- Inline comments: Only for non-obvious logic (rare)
- TypeScript: Minimal; types are self-documenting via strict mode
- Python: Google-style docstrings on public functions
- Section headers: Used to delineate major code blocks (e.g., `# -----------\n# Part A: Text similarity\n# -----------`)
## Function Design
- Most functions 10–50 lines; up to 150 lines for scheduler loop functions
- Each function has a single primary responsibility
- Helper functions (prefixed `_`) extracted for reusability
- TypeScript: Destructured object props in component signatures
- Python: Named parameters with type hints; optional params have defaults
- No positional-only args; use keyword args for clarity
- TypeScript: Explicit return type annotations on all functions
- Python: Type hints on all parameters and return values
- Use union types (Python: `X | None`, TS: `X | undefined`) for optional returns
- Data structures (dataclasses, Pydantic models) for complex returns
## Module Design
- TypeScript: Default export for page components and main exports; named exports for utilities
- Python: All public functions and classes exportable; private functions prefixed with `_`
- TypeScript: `lib/` directory contains `api.ts`, `types.ts`, `utils.ts` (no `index.ts` barrel)
- Python: `__init__.py` files exist but remain minimal; imports done explicitly from modules
- Frontend separates concerns: `components/`, `lib/` (API, types, utils), `app/` (pages)
- Backend separates by layer: `app/` (config, models, schemas), `services/` (business logic), `routes/` (endpoints), `clients/` (external integrations)
## Async Patterns
- Async/await preferred; minimal promises
- SWR hooks handle async data fetching with caching
- useCallback for memoizing event handlers
- Async/await throughout backend services and route handlers
- AsyncSession for database operations (SQLAlchemy asyncio)
- AsyncClient for HTTP requests (httpx)
- asyncio.create_task() for background scheduling
## Type Safety
- Strict mode enabled; all implicit `any` flagged as errors
- Interface types define component props, API responses, and domain models
- Import types using `import type { Type }` to prevent circular dependencies
- Type hints on all function signatures (PEP 484)
- `from __future__ import annotations` for forward references
- Pydantic for runtime validation of API payloads and config
- SQLAlchemy `Mapped` type hints for ORM columns
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Pattern Overview
- Dual-platform market comparison (Polymarket vs Kalshi)
- Async/await throughout with APScheduler background loops
- Multi-tier matching strategy (keyword similarity → LLM verification → embedding-based fallback)
- Real-time pricing snapshots with canonical probability aggregation
- Terminal-first UI: browser-based React dashboard consuming REST API
## Layers
- Purpose: HTTP endpoints for frontend consumption
- Location: `backend/routes/events.py`
- Contains: REST endpoints for event listing, detail retrieval, price history, health checks
- Depends on: Database layer, services layer
- Used by: Frontend React application via `fetch()` calls
- Purpose: Core business logic—market matching, pricing aggregation, scheduling
- Location: `backend/services/`
- Contains:
- Depends on: Data layer, client layer, external APIs (Claude, Polymarket, Kalshi)
- Used by: Scheduler background tasks, API routes
- Purpose: Platform-specific API integrations with market fetching and parsing
- Location: `backend/clients/`
- Contains:
- Depends on: httpx, cryptography libraries, app config
- Used by: Services layer (scheduler, matchers)
- Purpose: Database abstraction, ORM models, async session management
- Location: `backend/app/`
- Contains:
- Depends on: SQLAlchemy, PostgreSQL
- Used by: API layer, services layer
- Purpose: Terminal-optimized React dashboard
- Location: `frontend/src/`
- Contains:
- Depends on: Next.js 16, React 19, SWR, TailwindCSS
- Used by: End users via browser
## Data Flow
- Database: PostgreSQL via SQLAlchemy, primary source of truth
- Memory: Scheduler module-level `last_matching_run`, `last_pricing_run` timestamps
- Browser cache: SWR client-side caching of API responses
## Key Abstractions
- Purpose: Cross-platform market pair representation
- Examples: `backend/app/models.py` (ORM), `frontend/src/lib/types.ts` (TypeScript)
- Pattern: Single source of truth for matched market pairs with match confidence and resolution delta
- Purpose: Individual platform market record before matching
- Examples: `backend/app/models.py`
- Pattern: Deduplication by (platform, platform_id); marked as `is_matched` after successful match
- Purpose: Structured output of matching decision
- Examples: `backend/services/matcher.py` (dataclass)
- Pattern: Carries is_match, confidence, canonical_title, category, resolution_difference, tier
- Purpose: Historical price record for charting and trend analysis
- Examples: `backend/app/models.py` (ORM)
- Pattern: Immutable append-only records per event; indexed by event_id + timestamp
## Entry Points
- Location: `backend/app/main.py`
- Triggers: `uvicorn` start
- Responsibilities:
- Location: `frontend/src/app/page.tsx` (Next.js App Router)
- Triggers: Browser navigation to `/`
- Responsibilities:
- Function: `scheduler.matching_loop()` (async, no return)
- Triggers: APScheduler job, every 15 minutes
- Responsibilities:
- Function: `scheduler.pricing_loop()` (async, no return)
- Triggers: APScheduler job, every 30 seconds
- Responsibilities:
## Error Handling
## Cross-Cutting Concerns
- Approach: `logging` module, configured via settings.log_level
- Strategy: Info level for job start/completion, debug for internal details, warning/error for failures
- Examples: `logger.info("Fetched %d Polymarket markets", len(bulk))`
- Approach: Pydantic schemas on API response, inline checks for business logic
- Examples: Skip events where poly_yes_price < 0.01, kalshi_yes_price < 0.01
- LLM verification prompt includes: dates, resolution criteria equivalence checks
- Approach: Kalshi uses RSA request signing (key_id + private key); Polymarket uses API key (implicit)
- Configuration: `settings.kalshi_key_id`, `settings.kalshi_private_key_path`, loaded from .env
- Frontend: No auth required; API has CORS allow_origins=["http://localhost:3000"]
- Approach: None implemented; relies on platform API rate limits
- Risk: High-frequency polling (30sec pricing loop) on Kalshi/Polymarket may hit limits
<!-- GSD:architecture-end -->


<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [rohanthomas1202/EdgeTerminal](https://github.com/rohanthomas1202/EdgeTerminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
