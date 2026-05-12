## opencodeintel

> - NO emojis anywhere -- not in code, comments, docs, or commit messages

# CodeIntel Development Rules

## Code Style

### General
- NO emojis anywhere -- not in code, comments, docs, or commit messages
- Prefer files under 200 lines. Larger files allowed when logically cohesive.
- Comments explain WHY, not WHAT. Keep them brief.
- No decorative headers or ASCII art
- Casual tone in comments -- write like a human, not a robot

### JSDoc Policy
- JSDoc allowed for public API functions and exported hooks
- Keep JSDoc minimal -- document params and return types, not obvious behavior

### Large File Exceptions
These files exceed 200 lines but are approved due to logical cohesion:
- `backend/routes/playground.py`
- `backend/services/dna_extractor.py`
- `backend/services/dependency_analyzer.py`
- `frontend/src/components/DependencyGraph/GraphView/index.tsx`
- `frontend/src/components/DependencyGraph/MatrixView/index.tsx`

### Frontend (TypeScript/React)
- Package manager: **Bun only**. Never npm, never yarn.
- Always `bun install`, `bun run dev`, `bun run build`
- Never delete `bun.lock` or create `package-lock.json`
- Use shadcn/ui components over custom UI
- Tailwind for styling, no CSS files
- Functional components with hooks, no class components
  (ErrorBoundary is the only exception -- React requires class for error boundaries)
- Use React Query (`useQuery`) for all server data fetching, never raw fetch in useEffect
- Custom hooks go in `frontend/src/hooks/` with `use` prefix
- Shared types go in `frontend/src/types.ts`
- Keep components focused -- one concern per file

### React Patterns
- Derive state during render when possible, don't sync with useEffect
- Define components at module scope, never inside other components (causes remounts)
- Use stable keys (item.id), not array indices, for lists that change
- Colocate state with the component that uses it, lift only when needed
- Use `cn()` from `@/lib/utils` for conditional class merging

### Testing
- Vitest + happy-dom for unit/component tests
- Tests live in `frontend/src/test/` using `@/` alias imports
- Run with `bun run test`, runs in CI on every PR
- Test file naming: `ComponentName.test.tsx` or `hookName.test.ts`
- Smoke tests for critical paths: does it render, does the hook return correct data
- Backend tests: pytest in `backend/tests/`, run with `cd backend && pytest tests/ -v`

### Backend (Python)
- Python 3.11+ required
- Type hints on all function signatures
- Async/await for I/O operations
- PEP 8 style, max 120 char lines
- Use existing patterns from `services/` directory
- All new services should follow the singleton pattern used in `dependency_analyzer.py`
- Startup validation in `config/startup_checks.py` -- add new required/optional env vars there

### Commits
- Format: `type: description` (e.g., `fix: remove broken link`)
- Types: feat, fix, docs, refactor, test, chore
- No emojis in commit messages
- Keep commits focused -- one change per commit
- PR scope: one concern per PR. Don't mix bug fixes with features.

### Code Review
- All PRs go through CodeRabbit automated review
- Fix real findings (bugs, type safety, missing deps). Skip nitpicks unless trivial.
- PRs target `OpenCodeIntel/opencodeintel:main` from your fork

## Architecture

### Project Structure
```text
backend/                    # FastAPI, Python 3.11+
  config/                   # API config, startup checks
  middleware/               # Auth, rate limiting
  routes/                   # API endpoints (all use /api/v1/ prefix)
  services/                 # Business logic (singleton services)
  tests/                    # pytest test files
frontend/                   # React 18, TypeScript, Vite, Bun
  src/
    components/             # UI components (one concern per file)
      dashboard/            # Sidebar, TopNav, DashboardHome, RepoListView, RepoDetailView
      DependencyGraph/      # GraphView (Sigma.js), MatrixView (DSM), ImpactPanel
      ui/                   # shadcn/ui primitives
      landing/              # Marketing pages
      docs/                 # Documentation components
    hooks/                  # Custom React hooks (useCachedQuery, useRepos, etc.)
    contexts/               # React contexts (AuthContext)
    config/                 # API URL config, demo repos
    pages/                  # Route-level page components
    test/                   # Vitest test files
    types.ts                # Shared TypeScript interfaces
mcp-server/                 # MCP protocol server for Claude/Cursor
```

### API Versioning
- All endpoints use `/api/v1/` prefix
- Version config in `backend/config/api.py`

### Data Flow
- Frontend fetches via React Query hooks in `hooks/useCachedQuery.ts`
- Dual-layer caching: React Query (memory) + localStorage (persistence)
- Backend caches dependency graphs and DNA in Supabase
- Real-time indexing progress via WebSocket

### Key Services
- `indexer_optimized.py` -- Code parsing and embedding (tree-sitter + OpenAI/Voyage)
- `dependency_analyzer.py` -- Import graph extraction (tree-sitter AST)
- `style_analyzer.py` -- Convention detection
- `dna_extractor.py` -- Architectural pattern extraction + team rules detection
- `supabase_service.py` -- Database operations (uses service role key)
- `auth.py` -- JWT verification (uses anon key + JWT secret)

### Environment Variables
- Root `.env.example` is the source of truth for all env vars
- Docker passes vars via `docker-compose.yml` environment block
- Frontend vars must be prefixed with `VITE_`
- New required vars: add to `config/startup_checks.py` REQUIRED_VARS
- New optional vars: add to `config/startup_checks.py` OPTIONAL_VARS

## What NOT to Do

- Don't use npm or yarn (use Bun)
- Don't add emojis
- Don't write verbose comments
- Don't add "AI-looking" badges or decorations
- Don't define components inside other components
- Don't use raw fetch in useEffect (use React Query)
- Don't put inline types when a shared interface exists in types.ts
- Don't create PRs with multiple unrelated concerns
- Don't merge PRs without CI passing

---
> Source: [OpenCodeIntel/opencodeintel](https://github.com/OpenCodeIntel/opencodeintel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
