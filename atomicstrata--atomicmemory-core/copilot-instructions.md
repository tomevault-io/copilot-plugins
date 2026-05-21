## atomicmemory-core

> Open-source memory engine for AI applications. Docker-deployable backend with

# Atomicmemory-core

Open-source memory engine for AI applications. Docker-deployable backend with
semantic retrieval, AUDN mutation, and contradiction-safe claim versioning.

## Contributor Context

Use this file as the repo-local contributor guide. When a task requires
behavior from another AtomicMemory repo, rely on the public package/API contract
and document any assumption in the PR.

Before changing code, read the relevant local files in this repo first:

- `README.md` for user-facing setup and API usage.
- `package.json` for the supported scripts.
- `src/config.ts` before touching environment or runtime config.
- `openapi.yaml` / `openapi.json` when changing HTTP behavior.

## Architecture

- **Runtime**: Express server (TypeScript, ESM)
- **Storage**: Postgres + pgvector
- **Embeddings**: Pluggable (openai, openai-compatible, ollama, transformers/WASM)
- **Structure**: Single-package repo. Routes → Services → Repository → Postgres.

### Key modules
- `src/routes/` — Express route handlers
- `src/services/` — Business logic (ingest, search, AUDN, packaging)
- `src/db/` — Repository layer, schema, migrations
- `src/services/memory-service.ts` — Thin facade delegating to focused sub-modules
- `src/services/memory-ingest.ts` — Ingest pipeline
- `src/services/memory-search.ts` — Search/retrieval pipeline
- `src/services/memory-audn.ts` — AUDN mutation decisions
- `src/services/memory-crud.ts` — List/get/delete/consolidate
- `src/services/memory-storage.ts` — Canonical fact storage and projections

### Boundary rule
This repo is the **releaseable runtime engine**. Eval harnesses, benchmarks,
competitive analysis, and design explorations do not belong here unless they
are part of supported product behavior. If it changes shipped backend behavior,
it belongs here. If it only changes benchmark outputs or scoring methodology,
keep it outside this repo.

See https://docs.atomicmemory.ai/platform/consuming-core for the stable seams
(HTTP, in-process runtime container, docker/E2E compose) that SDK consumers and
internal tools should use.

## Local Setup

Use Node.js 22 or newer. Install dependencies from this repo root:

```bash
npm install
```

Core tests that touch Postgres require `.env.test` with a reachable Postgres
database that has pgvector available. Runtime commands load env explicitly via
the npm scripts; do not bypass those scripts with raw `tsx`/`node` commands
unless you also load the correct env file.

## Development Guidelines

### Code Style & Standards
- Code files (TypeScript, JavaScript, shell, SQL, Python) must be smaller than 400 lines excluding comments. Refactor when exceeded. **Markdown and other prose/config files (`.md`, `.mdx`, `.yaml`, `.json`, `.toml`) are exempt.**
- Functions must be smaller than 40 lines excluding comments and catch/finally blocks.
- Test files must be smaller than 400 lines. Tests must be smaller than 40 lines.
- Use TypeScript with proper types for all variables, parameters, and return values.
- Use interfaces for object shapes. Avoid `any`. Use generics when appropriate.
- Use optional chaining (`?.`) and nullish coalescing (`??`).
- No fallback modes — if something fails, fail closed, don't run degraded.
- No silent error catching — all errors must be logged or propagated.
- No direct access to env vars (use `src/config.ts`).
- No hardcoded values — use named constants.
- No timing-based solutions in code or tests. All solutions must be deterministic.

### Clean Code Rules
- Meaningful names that reveal purpose.
- One function, one responsibility.
- Avoid magic numbers — use named constants.
- Keep code DRY. Duplicate code means duplicate bugs.
- Avoid deep nesting — flatten control flow.
- Comment why, not what.
- Limit function arguments — group related data into objects.

### Comments and Documentation
- Include a JSDoc comment at the top of each file.
- Write clear comments for complex logic.
- Document public APIs and functions.
- Keep comments up-to-date with code changes.

### Pre-Commit Checks

Before committing any work:

1. `npx tsc --noEmit` — type-check passes
2. `npm test` — all tests pass (requires Postgres via .env.test)
3. `fallow --no-cache` — zero issues (dead code, duplication, complexity). Always use `--no-cache` before committing to match CI behavior. Use `fallow fix --dry-run` to preview, `fallow fix --yes` to apply. Fix remaining issues manually.
4. `npm run build` — build succeeds when TypeScript, exports, or package output changed.
5. `npm run check:openapi` — run when route schemas, request/response shapes, or OpenAPI metadata changed.

### Running Tests
- Full suite: `npm test` (requires DATABASE_URL in .env.test pointing to Postgres with pgvector)
- Single test: `dotenv -e .env.test -- npx vitest run "src/**/__tests__/<path>" --reporter verbose`
- Deployment tests: `npm run test:deployment`
- Docker smoke: `npm run test:docker-smoke`

### Git Workflow
- Never commit directly to main. Always create a branch.
- When committing, create a temporary `commit-message.txt` and use `-F`, then delete.
- When creating a PR, create a temporary `pr-description.md`, use `gh pr create --body-file`, then delete.
- Never use `git reset --hard`.

## General Rules

1. Read the codebase before making changes. Never speculate about code you haven't opened.
2. Check in with the user before making major changes.
3. Make every change as simple as possible — minimal impact, maximum clarity.
4. Check for existing implementations before writing new code.
5. Mutations must fail closed — no silent fallback-to-ADD on UPDATE/DELETE/SUPERSEDE errors.

---
> Source: [atomicstrata/atomicmemory-core](https://github.com/atomicstrata/atomicmemory-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
