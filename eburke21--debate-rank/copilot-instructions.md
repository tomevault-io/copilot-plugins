## debate-rank

> DebateRank is a multi-criteria LLM-as-judge evaluation platform. Users submit debate arguments, 4 LLM judges (Logic, Evidence, Persuasion, Originality) score each argument, and a leaderboard ranks them with configurable weight sliders. Built as a portfolio project demonstrating AI evaluation infrastructure.

# CLAUDE.md

## Project Overview

DebateRank is a multi-criteria LLM-as-judge evaluation platform. Users submit debate arguments, 4 LLM judges (Logic, Evidence, Persuasion, Originality) score each argument, and a leaderboard ranks them with configurable weight sliders. Built as a portfolio project demonstrating AI evaluation infrastructure.

## Architecture

Monorepo with two isolated packages communicating over HTTP:

- **Backend** (`backend/`): Python 3.12+, FastAPI, SQLAlchemy async, PostgreSQL, Anthropic SDK
- **Frontend** (`frontend/`): React 19, TypeScript, Vite, Chakra UI v3, Recharts

Orchestrated via Docker Compose (db, backend, frontend services).

## Quick Reference

### Start everything
```bash
docker compose up --build
```
- Backend: http://localhost:8000
- Frontend: http://localhost:5173
- Health check: `curl http://localhost:8000/api/health`

### Backend commands (run from `backend/`)
```bash
uv sync                          # Install dependencies
uv run uvicorn app.main:app --reload  # Run dev server standalone
uv run ruff check .              # Lint
uv run ruff format --check .     # Format check
uv run ruff format .             # Auto-format
uv run pytest tests/ -v          # Run all tests (58 total, requires running Postgres)
uv run pytest tests/test_schemas.py -v    # Schema validation tests only (17, no DB needed)
uv run pytest tests/test_sanitization.py -v  # Sanitization unit tests (5, no DB needed)
uv run pytest tests/test_scoring.py -v   # Composite scoring unit tests (6, no DB needed)
uv run pytest tests/test_database.py -v   # DB integration tests (2, requires Postgres)
uv run pytest tests/test_api.py -v        # API integration tests (14, requires Postgres)
uv run pytest tests/test_judge.py -v      # Judge system tests (14, requires Postgres for orchestrator)
```

### Alembic migrations (run from `backend/`, requires DATABASE_URL)
```bash
DATABASE_URL=postgresql+asyncpg://debaterank:localdev@localhost:5432/debaterank \
  uv run alembic upgrade head                        # Apply all migrations
DATABASE_URL=postgresql+asyncpg://debaterank:localdev@localhost:5432/debaterank \
  uv run alembic revision --autogenerate -m "desc"   # Generate migration from model changes
DATABASE_URL=postgresql+asyncpg://debaterank:localdev@localhost:5432/debaterank \
  uv run alembic current                             # Check current migration state
```
Note: Alembic runs synchronously — `env.py` swaps `+asyncpg` → `+psycopg2` automatically. The `psycopg2-binary` dev dependency exists for this purpose.

### Seed script (run from `backend/`, requires DATABASE_URL)
```bash
uv run python -m scripts.seed                    # Full seed with LLM evaluation (needs ANTHROPIC_API_KEY)
uv run python -m scripts.seed --dry-run           # Preview without DB or API calls
uv run python -m scripts.seed --skip-evaluation   # Insert rows without judging (DB only)
uv run python -m scripts.seed --reset             # Delete all data, then re-seed
uv run python -m scripts.seed --reset --skip-evaluation  # Fresh insert without judging
```
Expected runtime: ~2-3 min for 12 arguments. Expected API cost: ~$0.10-0.30.

### Frontend commands (run from `frontend/`)
```bash
npm install                      # Install dependencies
npm run dev                      # Run dev server
npm run build                    # Type-check + build
npm run lint                     # ESLint
npx tsc --noEmit                 # Type-check only
npm test                         # Run all tests (15 total, Vitest)
npm run test:watch               # Watch mode for development
```

## Code Conventions

### Backend (Python)
- **Package manager:** uv (not pip). Always use `uv run` to execute commands.
- **Linting:** Ruff with rules E, F, I, UP. Target Python 3.12. Line length 100.
- **Testing:** pytest with `asyncio_mode = "auto"` — no need for `@pytest.mark.asyncio`.
- **Config:** All settings via `pydantic-settings.BaseSettings` in `app/config.py`. Environment variables, not hardcoded values.
- **DB driver:** `asyncpg` via `postgresql+asyncpg://` connection strings. All database access is async.
- **SQLAlchemy models:** Use `server_default=text(...)` for DB-generated defaults (UUIDs, timestamps, status). Use `TZDateTime = DateTime(timezone=True)` alias for timestamp columns (not `TIMESTAMPTZ` — SQLAlchemy doesn't export it).
- **Barrel exports:** `app/models/__init__.py` must import all models so `Base.metadata` is fully populated for Alembic autogenerate. Same pattern in `app/schemas/__init__.py`.
- **Enums:** Use `StrEnum` (not `(str, Enum)`) — required by Ruff UP042 for Python 3.12 target.
- **Imports:** Use absolute imports from `app.*` (e.g., `from app.models import Topic`).
- **Models vs Schemas:** SQLAlchemy models in `app/models/`, Pydantic request/response models in `app/schemas/`. Don't mix them.
- **Route handlers:** Thin translation layer — validate input (Pydantic), query (SQLAlchemy with `selectinload`), transform (ORM → schema), respond. No business logic in routes; sanitization in `services/`, rate limiting in `middleware/`.
- **Eager loading:** Always use `selectinload(Argument.scores)` when querying arguments that need scores. Lazy loading fails in async context (`MissingGreenlet`).
- **Error envelope:** All errors use `{"error": {"code": "...", "message": "...", "fields": {...}}}`. Validation errors (422) are reshaped from Pydantic's format via a global `RequestValidationError` exception handler.
- **Ruff excludes:** `alembic/versions/` is excluded from linting (auto-generated migration code).
- **LLM judge calls:** Use `AsyncAnthropic` from the `anthropic` SDK. Judge functions accept an optional `client` parameter for test injection; production creates a default client per call (SDK manages connection pooling internally).
- **Background tasks:** `BackgroundTasks` callbacks run after the response is sent, outside the request's DI scope. They must create their own DB session via `async_session()`, not reuse the request's session.
- **Timestamps:** Use `datetime.now(UTC)` (not deprecated `datetime.utcnow()`). Import `UTC` from `datetime`.
- **Latency measurement:** Use `time.monotonic()` for duration calculations, not `time.time()` (immune to clock adjustments).

### Frontend (TypeScript/React)
- **UI library:** Chakra UI v3 — uses `createSystem()` / `defineConfig()`, NOT v2's `extendTheme()`.
- **Rubric colors:** Defined as semantic tokens in `src/theme/index.ts` — use `rubric.logic`, `rubric.evidence`, `rubric.persuasion`, `rubric.originality` instead of raw color values.
- **HTTP client:** axios (configured in `src/api/client.ts`). Base URL from `VITE_API_URL` env var (defaults to `http://localhost:8000`).
- **Charts:** Recharts for data visualization.
- **Pure function extraction:** `redistributeWeights()` and `computeComposite()` are exported as standalone pure functions from `hooks/useWeights.ts` — testable without React, reusable outside the hook (e.g., `Leaderboard.tsx` imports `computeComposite` directly).
- **Derived state:** Use `useMemo` for computed values like weighted rankings — never `useEffect` + `setState` (causes double-render flicker).
- **Framer Motion:** `motion.div` with `layout` prop for animated row reordering. `LayoutGroup` coordinates siblings, `AnimatePresence` handles enter/exit. Use `mode="wait"` for element swaps (spinner→score), `mode="popLayout"` for list reordering. Keyframe arrays on `animate` for one-shot pulse effects.
- **Chakra v3 Slider API:** Compound components `Slider.Root` → `Slider.Control` → `Slider.Track` → `Slider.Range` + `Slider.Thumb`. Uses `onValueChange` (not `onChange`); value is always an array `[number]` even for single-thumb sliders.
- **Chakra v3 Drawer API:** `Drawer.Root` → `Portal` → `Drawer.Backdrop` + `Drawer.Positioner` → `Drawer.Content`. Props: `open` (not `isOpen`), `onOpenChange={(e) => { if (!e.open) onClose() }}` (not `onClose`), `placement="end"`, `size="md"`.
- **Chakra v3 Dialog API:** `Dialog.Root` → `Portal` → `Dialog.Backdrop` + `Dialog.Positioner` → `Dialog.Content`. Same prop pattern as Drawer. Renamed from v2's `Modal`.
- **Chakra v3 Collapsible:** `Collapsible.Root` → `Collapsible.Trigger` → `Collapsible.Content`. Use `unmountOnExit` for performance. `asChild` on Trigger merges behavior into child element (no wrapper DOM node).
- **Responsive design:** Chakra's responsive object syntax `display={{ base: "none", md: "block" }}` for breakpoint-based visibility.
- **`useReducer` for related state:** When multiple state values change together (e.g., polling state: status, scores, composite, isComplete, error), use `useReducer` instead of multiple `useState` calls. Batches updates into one render; `dispatch` has stable identity; satisfies ESLint `set-state-in-effect` rule.
- **Component state machines:** Use a single `Phase` union type (`"drafting" | "scoring" | "complete"`) instead of multiple boolean flags. Eliminates impossible states. Phase transitions during render (not in `useEffect`) skip stale DOM commits.
- **Polling pattern:** `setInterval` + `Date.now()` wall-clock timeout. Transient errors swallowed (next tick retries). Three cleanup paths: unmount (effect cleanup), terminal status (poll callback), disabled (effect early return).
- **Prop drilling:** Deliberate choice — still works at Phase 7's depth. Each component uses its props (no pass-through intermediaries). Context would earn its keep only if overlays needed shared state.
- **Chakra v3 Color Mode:** No `useColorMode()` hook in v3. Custom `useColorMode` hook in `hooks/useColorMode.ts` manages state + localStorage persistence + system preference detection. Wrapped via `<Theme appearance={appearance}>` (renders a `<div>`, add `minH="100vh"`). Semantic tokens (`bg`, `fg`, `fg.muted`) auto-respond; hardcoded colors need `{ base: "red.50", _dark: "red.950" }` overrides.
- **Chakra v3 Toaster:** `createToaster()` returns a module-level store (not a hook) — callable from non-React code like axios interceptors. `<Toaster>` component in `main.tsx` requires a children render function wrapping `<Toast.Root>`.
- **Error toasts:** Centralized via axios response interceptor in `api/client.ts`. Handles network errors, timeouts, 429, 422 (extracts fields from error envelope), 5xx. Interceptor re-rejects so components can also show inline errors.
- **Frontend testing:** Vitest with jsdom environment, configured in `vite.config.ts`. Setup file at `src/test/setup.ts` imports `@testing-library/jest-dom/vitest`. Pure functions tested directly (no `renderHook` needed).

## Project Structure

```
backend/
  app/
    config.py          # pydantic-settings config (DATABASE_URL, ANTHROPIC_API_KEY, etc.)
    main.py            # FastAPI app, CORS, routers, middleware, exception handler
    database.py        # Async engine, session factory, Base, get_db dependency
    models/            # SQLAlchemy models (Topic, Argument, Score) + barrel __init__.py
    schemas/           # Pydantic request/response models + enums + error envelope + barrel __init__.py
    api/               # Route handlers: health.py, topics.py, arguments.py
    services/          # Business logic: sanitization.py, judge.py (LLM evaluation), scoring.py (weighted composite)
    middleware/        # rate_limit.py — IP-based, POST /api/arguments only
  data/
    seed_arguments.json # 12 curated arguments targeting diverse rubric profiles
  scripts/
    seed.py            # Seed pipeline: load JSON, insert, evaluate, report (--dry-run, --reset, --skip-evaluation)
  alembic/
    env.py             # Customized: swaps asyncpg→psycopg2, imports Base from app.models
    versions/          # Auto-generated migration files (excluded from ruff)
  tests/
    conftest.py        # Fixtures: db_session (rollback), client (httpx+DI override), active_topic, scored_argument, no_active_topics
    test_schemas.py    # 17 Pydantic validation tests
    test_sanitization.py # 5 sanitization unit tests
    test_database.py   # 2 integration tests (insert chain, cascade delete)
    test_api.py        # 14 API integration tests (CRUD, validation, rate limiting)
    test_judge.py      # 14 judge tests (output parsing, single judge, orchestrator)
    test_scoring.py    # 6 composite weighted score tests

frontend/src/
  theme/index.ts     # Chakra UI v3 system + rubric color tokens
  api/client.ts      # Axios API client (4 typed fetch functions) + error toast interceptor
  types/index.ts     # TypeScript types mirroring backend schemas + RUBRICS array + RUBRIC_META
  test/setup.ts      # Vitest setup — jest-dom matchers
  hooks/
    useWeights.ts    # Weight redistribution + computeComposite (pure functions + React hook)
    useWeights.test.ts # 15 unit tests for redistribution + composite scoring
    useLeaderboard.ts # Topic + arguments data fetching
    useArgumentPolling.ts # useReducer-based polling (2s interval, 30s timeout, transient error resilience)
    useColorMode.ts  # Dark/light mode with localStorage persistence + system preference detection
  components/
    toaster.ts               # Shared Chakra v3 toaster instance (module-level, not a hook)
    ColorModeButton.tsx      # Sun/moon emoji toggle with aria-label
    TopicHeader.tsx          # Topic title, description, argument count badge
    WeightSlider.tsx         # Single Chakra v3 compound slider with rubric label
    WeightSliderPanel.tsx    # 4 sliders + reset button + animated distribution bar
    LeaderboardRow.tsx       # Rank badge + body preview + mini score bars + weighted score + layout animation + highlight
    Leaderboard.tsx          # Weighted sort via useMemo + AnimatePresence + loading skeletons
    ScoreRadarChart.tsx      # Recharts RadarChart with 4 rubric axes (0-10), ResponsiveContainer
    JudgeCard.tsx            # Rubric score + collapsible rationale (Chakra v3 Collapsible)
    ArgumentDetailDrawer.tsx # Chakra v3 Drawer — full text, radar chart, judge cards, partial score handling
    SubmitArgumentModal.tsx  # 3-phase state machine (drafting→scoring→complete), char counter, Chakra v3 Dialog
    ScoringProgress.tsx      # 4 judge indicators with AnimatePresence spinner→score transitions
```

## Key Design Decisions

- **4 rubrics:** Logic, Evidence, Persuasion, Originality — each scored 1-10 by a separate LLM judge call
- **Composite scoring:** Weighted average using user-adjustable sliders (recomputed client-side)
- **Argument body:** 50-2000 characters (validated in both Pydantic schema and DB CHECK constraint — defense in depth)
- **Score uniqueness:** One score per (argument_id, rubric) pair enforced by UNIQUE constraint
- **Server-side defaults:** UUIDs (`gen_random_uuid()`), timestamps (`now()`), and status (`'pending'`) generated by PostgreSQL via `server_default=text(...)` — not Python-side
- **Two response shapes:** `ArgumentSummary` uses `dict[Rubric, int]` for leaderboard (efficient client-side weight math); `ArgumentDetail` uses `list[ScoreResponse]` for detail view (includes rationales)
- **Test isolation:** Transaction rollback pattern — each test runs in a transaction that rolls back, no data persists, no cleanup needed
- **Alembic on startup:** `docker-compose.yml` backend command runs `alembic upgrade head` before starting uvicorn
- **Rate limiting:** In-memory, IP-based, 10/min on `POST /api/arguments` only. Sliding window with timestamp pruning. Scoped via path+method check in middleware.
- **Input sanitization:** `services/sanitization.py` — strip whitespace, strip HTML tags (regex), collapse 3+ newlines to 2, normalize unicode (NFC). No external dependencies.
- **LeaderboardResponse wrapper:** `GET /api/arguments` returns `{topic: TopicResponse, arguments: list[ArgumentSummary]}` — not a bare list. Composite shape matching spec §5.2.
- **API test pattern:** `httpx.AsyncClient` with `ASGITransport` + `app.dependency_overrides[get_db]` to swap in rollback session. Tests go through full ASGI stack (middleware, routing, serialization).
- **Judge model:** `claude-sonnet-4-20250514` with 30-second per-call timeout via `asyncio.wait_for()`.
- **Parallel judge dispatch:** `asyncio.gather(*tasks, return_exceptions=True)` — 4 judges run concurrently, partial failures produce partial results instead of total failure.
- **Retry on parse failure:** If a judge's response isn't valid JSON, a correction prompt (with the error and original response) is sent once. Two consecutive failures raise `ValueError`.
- **Status state machine:** `pending` → `scored` (4/4 succeed), `partial` (2-3/4), or `failed` (0-1/4). Composite score is the mean of successful scores (None if failed).
- **Background task dispatch:** `POST /api/arguments` returns 202 immediately. Judge evaluation runs as a `BackgroundTasks` callback with its own DB session.
- **Judge test isolation:** Orchestrator tests use `monkeypatch.setattr("app.services.judge.evaluate_single_judge", mock)` to control per-rubric results. Single judge tests use injectable `client` parameter with `AsyncMock`.
- **Seed data design:** 12 hand-crafted arguments with deliberate asymmetry across rubrics — each argument targets a specific archetype (data-heavy, vivid anecdote, contrarian take, etc.) so weight sliders produce visible ranking changes. Scores are live-evaluated, not hardcoded.
- **Seed script invocation:** Run as `uv run python -m scripts.seed` (not `python scripts/seed.py`) so `app.*` imports resolve correctly. `scripts/__init__.py` exists for this purpose.
- **Weight redistribution:** "Last item gets the remainder" pattern ensures `sum(weights) === 100` — `Math.round()` on all-but-last rubric, last rubric = `remaining - distributed`. Avoids rounding drift across 3 proportional calculations.
- **Client-side composite scoring:** Weighted average recomputed in `useMemo` on every weight change — no backend round-trip. Only rubrics with non-null scores contribute (handles partial-status arguments).
- **Animated leaderboard reordering:** Framer Motion's `layout` prop on each row enables spring-based positional animation (300ms) when sort order changes from weight adjustments.
- **Distribution bar:** CSS `flex={weight}` on colored segments provides proportional visualization of current weights; `transition="flex 0.3s ease"` animates segment size changes.
- **Argument detail on demand:** `ArgumentDetailDrawer` fetches `GET /api/arguments/{id}` when opened (not preloaded). Resets state on close so stale data doesn't flash on reopen.
- **Submit modal state machine:** 3-phase `drafting → scoring → complete`. Single `Phase` union type prevents impossible states. Modal close prevented during scoring phase (`handleClose` early-returns). Phase transition from scoring → complete happens during render (not `useEffect`) to skip stale DOM commits.
- **Character counter UX:** Color-coded feedback: gray (0-49, "X more needed"), green (50-1800), yellow (1800-2000), red (>2000). Submit button disabled until 50-2000 chars.
- **Polling for scoring progress:** `useArgumentPolling` hook uses `useReducer` + `setInterval` (2s) + `Date.now()` timeout (30s). Swallows transient errors (next tick retries). Stops on terminal status (`scored`, `partial`, `failed`). Returns `initialState` when disabled (derived state, no effect-based reset).
- **New argument highlight:** After submission, `highlightedArgumentId` triggers blue border + Framer Motion keyframe pulse on the matching `LeaderboardRow`. Cleared after 2 seconds via `setTimeout`.
- **Derived open state:** `!!selectedArgumentId` derives drawer's `isOpen` from a single source of truth. Avoids synchronization bugs between parallel boolean flags.
- **Docker `.venv` exclusion:** Backend service in `docker-compose.yml` uses `- /app/.venv` anonymous volume to prevent WatchFiles reload storm from `uv sync` inside container. Same pattern as `- /app/node_modules` for frontend.
- **Test fixture isolation with seed data:** Transaction rollback pattern sees committed seed rows. Fixtures that need "no active topic" must explicitly deactivate existing topics within the transaction. `no_active_topics` fixture handles this; `active_topic` fixture deactivates before creating.
- **3-tier CI pipeline:** GitHub Actions with 3 parallel jobs: backend (Python + Postgres service), frontend (Node + Vitest + build), docker (compose build). Each catches different failure categories; total CI ≈ max(job times), not sum.
- **Architecture Decision Records:** 4 ADRs in `docs/architecture.md` — client-side ranking, parallel judge dispatch, PostgreSQL over SQLite, structured JSON output. Context → Decision → Consequences → Alternatives Considered format.

## API Endpoints

| Method | Path | Status | Response |
|--------|------|--------|----------|
| `GET` | `/api/health` | 200 | `{"status": "healthy", "version": "0.1.0"}` |
| `GET` | `/api/topics/active` | 200 / 404 | `TopicResponse` with argument count |
| `GET` | `/api/arguments?topic_id=` | 200 / 404 | `LeaderboardResponse` (topic + scored arguments with `dict` scores) |
| `GET` | `/api/arguments/{id}` | 200 / 404 | `ArgumentDetail` (scores as `list` with rationales) |
| `POST` | `/api/arguments` | 202 / 422 / 429 | `ArgumentSubmitResponse` — creates argument, dispatches judge evaluation in background |

## Spec & Planning Docs

- `dev-docs/spec.md` — Full technical specification
- `dev-docs/plan.md` — 8-phase implementation plan
- `dev-docs/progress.md` — Implementation log with task details and commit messages
- `dev-docs/key-learnings.md` — Conceptual learnings and insights by phase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eburke21)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eburke21)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
