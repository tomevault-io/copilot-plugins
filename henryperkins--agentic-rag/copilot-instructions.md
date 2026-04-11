## agentic-rag

> - `backend/src/` hosts the Fastify API, orchestration services, and retrieval agents; extend existing layer directories instead of creating new ones.

# Repository Guidelines

## Project Structure & Module Organization
- `backend/src/` hosts the Fastify API, orchestration services, and retrieval agents; extend existing layer directories instead of creating new ones.
- `backend/scripts/` centralizes database setup, migrations, and ingestion utilities, and `backend/tests/` mirrors service names with Vitest suites.
- `frontend/src/` plus `shared/types.ts` provide the Vite UI and shared SSE contracts, while `samples/` holds corpus fixtures for local testing.

## Build, Test & Development Commands
- `docker compose up -d` boots PostgreSQL with pgvector/pg_trgm before any backend process.
- `npm install` restores all workspace dependencies; rerun after editing a `package.json`.
- `npm run db:setup && npm run db:migrate` create extensions and apply migrations in order.
- `npm run ingest:samples` loads the baseline corpus for smoke tests.
- `npm run dev` starts Fastify and Vite together (`http://localhost:3000` and `http://localhost:5173`).
- `npm run typecheck` and `npm run test` enforce TypeScript contracts and execute the backend Vitest suite.
- SQL agent scaffolding now follows a plan→binder pipeline. Flip `ENABLE_SQL_AGENT=true` in `backend/.env` and use `npm -w backend run test -- sqlAgent` or `-- sqlBinder` while iterating on `sql.planner.ts` / `sql.binder.ts`.

## Coding Style & Naming Conventions
- Stick to TypeScript ES modules, 2-space indentation, and explicit return types for services to keep agent boundaries predictable.
- Use `camelCase` for functions, `PascalCase` for React components or classes, and `UPPER_SNAKE_CASE` for environment constants.
- House UI logic under `components/` or `hooks/` with `.tsx` extensions, and annotate each server module with a brief layer comment.
- Frontend typography should rely on the shared CSS tokens in `frontend/src/styles.css` (e.g., `--type-body-line-height`, `--interactive-min-size`) to stay compliant with WCAG 2.2 spacing rules and 24×24 mobile target sizes.
- Validate external input using `zod` schemas in routes/config modules before invoking downstream agents.

## Testing Guidelines
- Place backend specs in `backend/tests/` using the `*.test.ts` naming pattern; favor scenario-driven assertions over exhaustive mocks.
- Toggle `MOCK_OPENAI=1` in `backend/.env` for deterministic runs, and add fixtures under `samples/` when expanding coverage.
- Import shared types when asserting SSE payloads, and include hybrid-weight expectations whenever retrieval logic shifts.

## Commit & Pull Request Guidelines
- Git history is unavailable here, so follow Conventional Commits (`feat:`, `fix:`, `chore:`) with a single scope per change.
- PR descriptions should list affected architecture layers, executed tests, and any new configuration toggles.
- Attach UI screenshots or curl transcripts for behavior changes, and update README/DEPLOYMENT notes when workflows shift.

## Security & Configuration Tips
- Copy `backend/.env.example` to `backend/.env`, populate `DATABASE_URL` and API keys, and keep secrets out of version control.
- Constrain `env.CORS_ORIGIN` to trusted domains and leave rate-limiting/auth hooks active unless intentionally testing failures.
- Confirm migrations ran successfully before ingesting documents; stale schemas typically surface as retrieval errors.

## Web Search Agent (Layer 7)

**Trigger:** Recency indicators in query or empty local results
**Source:** OpenAI hosted `web_search` tool (GA)
**Location Awareness:** Optional approximate location (city/region/country/timezone)
**Domain Filtering:** Restrict to up to 20 allowed domains for trusted sources
**Citation Extraction:** Automatic inline citations with URLs, titles, and relevance scores
**Metadata:** Includes search query, domains searched, and all sources consulted
**Normalization:** Extracts domain as `document_id`, creates SHA-1 hash IDs
**Caching:** Bypassed for freshness (retrieval cache disabled when web active)

### New Features (2025 Upgrade)

1. **GA Tool (`web_search`)**: Upgraded from `web_search_preview` to generally available tool
2. **Citation Annotations**: Automatic extraction of inline citations with:
   - `citationStart` / `citationEnd`: Exact position in response text
   - `title`: Page title from OpenAI's search
   - `url`: Source URL
   - `relevance`: Position-based relevance score
3. **Search Metadata**: Tracks comprehensive search information:
   - `searchQuery`: Actual query sent to search (may differ from user query)
   - `domainsSearched`: Domains that were queried
   - `allSources`: Complete list of URLs consulted (beyond just citations)
4. **Domain Filtering**: Configure `WEB_SEARCH_ALLOWED_DOMAINS` to restrict to trusted sources:
   ```bash
   # Example: Only search academic and government sites
   WEB_SEARCH_ALLOWED_DOMAINS=wikipedia.org,nih.gov,edu
   ```
5. **Enhanced Transparency**: Frontend can display:
   - Inline citations with clickable links
   - "Sources consulted" section showing all URLs
   - Domain filtering status

### Configuration

```bash
# In backend/.env
ENABLE_WEB_SEARCH=true
WEB_SEARCH_CONTEXT_SIZE=medium  # low | medium | high
WEB_SEARCH_ALLOWED_DOMAINS=wikipedia.org,github.com,stackoverflow.com

# Optional location hints
WEB_SEARCH_CITY=San Francisco
WEB_SEARCH_REGION=California
WEB_SEARCH_COUNTRY=US
WEB_SEARCH_TIMEZONE=America/Los_Angeles
```

### API Response Format

```typescript
{
  chunks: [
    {
      title: "Page Title",
      url: "https://example.com/page",
      snippet: "Relevant excerpt from page",
      score: 1.0,
      relevance: 1.0,
      citationStart: 123,  // NEW
      citationEnd: 456     // NEW
    }
  ],
  metadata: {  // NEW
    searchQuery: "latest AI developments",
    domainsSearched: ["example.com", "wikipedia.org"],
    allSources: [
      "https://example.com/page1",
      "https://example.com/page2",
      "https://wikipedia.org/AI"
    ]
  }
}
```

## Query Classifier (Layer 2) - LLM Mode

**NEW:** The classifier now supports LLM-based intelligent routing alongside the existing heuristic mode.

### Configuration
```bash
# In backend/.env
USE_LLM_CLASSIFIER=true   # Enable LLM routing (default: false)
```

### Modes

**Heuristic Mode** (default, `USE_LLM_CLASSIFIER=false`):
- Fast, zero-cost keyword-based routing
- Detection rules:
  - **SQL**: Keywords like SELECT, COUNT, WHERE, GROUP BY, JOIN
  - **Web**: Recency indicators (latest, today, 2024, 2025, current, news, recent)
  - **Complexity**: Word count + operator keywords (join, compare, why, how, aggregate)
- Trade-off: May miss domain-specific jargon or complex intent

**LLM Mode** (`USE_LLM_CLASSIFIER=true`):
- Intelligent routing via GPT-4o-mini with structured JSON output
- Context-aware classification beyond simple keywords
- Automatic fallback to heuristics on error/timeout
- Cost: ~$0.0001 per query (~$100/month at 1M queries)

### Output Format
```typescript
{
  mode: "retrieve" | "direct",        // Retrieve docs or answer directly
  complexity: "low" | "medium" | "high",
  targets: ["vector", "sql", "web"]   // Which retrieval sources to use
}
```

### When to Use LLM Mode

✅ Enable when:
- Users ask questions with creative or ambiguous phrasing
- Domain-specific terminology not in keyword lists
- Classification accuracy > speed trade-off acceptable
- Multi-modal retrieval needs (vector + sql + web)

❌ Stick to heuristic when:
- Sub-50ms routing latency required
- Cost optimization is priority
- Query patterns are predictable
- Keyword matching suffices

### Testing
```bash
npm -w backend run test -- classifier.test.ts
```

Tests cover:
- Heuristic rule validation (24 test cases)
- LLM structured output parsing
- Fallback mechanism
- Target selection (vector/sql/web)
- Complexity assessment
- Mode selection (retrieve vs direct)

### Monitoring
Watch SSE logs for classification decisions:
```json
{
  "type": "agent_log",
  "role": "planner",
  "message": "Route: retrieve, complexity: medium, targets: vector+web",
  "ts": 1234567890
}
```

### Related Files
- `backend/src/services/orchestration/classifier.ts` - Classification logic
- `backend/src/config/env.ts` - Configuration
- `backend/tests/classifier.test.ts` - Test suite
- `CLAUDE.md` - Full system documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henryperkins)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/henryperkins)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
