## baruch

> Baruch is a Cloudflare Worker that powers an AI configuration assistant and knowledge base guide. Admins chat with Baruch to configure bt-servant-worker (prompt overrides, modes, MCP servers) via admin API tools. Baruch also uses MCP tools directly to help users access connected knowledge bases.

# Baruch — AI Configuration Assistant for BT Servant

## Project Overview

Baruch is a Cloudflare Worker that powers an AI configuration assistant and knowledge base guide. Admins chat with Baruch to configure bt-servant-worker (prompt overrides, modes, MCP servers) via admin API tools. Baruch also uses MCP tools directly to help users access connected knowledge bases.

## Architecture

- **Runtime**: Cloudflare Workers with single unified Durable Object (UserDO)
- **Framework**: Hono router
- **AI**: Claude via raw `globalThis.fetch()` to Anthropic API (SDK kept for types only)
- **Storage**: KV for prompt overrides, DO storage for user sessions/queues/memory
- **Streaming**: SSE-first — `/enqueue` returns SSE stream directly; callback mode also supported
- **Queue**: Internal FIFO within UserDO; processing in `fetch()` handler, `alarm()` as safety net only

## Key Conventions

### Never Do

- Never push directly to main without asking
- Never run `wrangler deploy` directly — all deploys through CI/CD
- Never merge a PR without user permission
- Never commit secrets or `.dev.vars`
- Never let security lint warnings (e.g. `security/detect-object-injection`) slide — fix the code or add an `eslint-disable-next-line` with a justification comment before merging

### Code Quality (Fitness Functions)

- **max-lines-per-function**: 50 (skip blanks/comments)
- **max-statements**: 25 per function
- **complexity**: 10 (cyclomatic)
- **max-depth**: 4 (nested blocks)
- **max-nested-callbacks**: 3
- **max-params**: 5

### Architecture Rules

- Types layer (`src/types/`) has zero internal dependencies
- Services cannot depend on routes
- No circular dependencies
- Run `pnpm architecture` to verify

### Testing

- Unit tests in `tests/unit/`
- E2E tests in `tests/e2e/`
- E2E tests skip on Windows (SQLite/workerd incompatibility)
- Run `pnpm test` before committing

### Code Review Policy

- After opening or updating a PR, always run the claude-code-review agent
- Fix all issues found by the reviewer, including low-severity ones
- After fixing, re-run the reviewer again
- Keep iterating (fix → re-review) until the reviewer finds zero issues

### Prompt System

Baruch has 5 admin-configurable prompt slots:

- `identity` — Who Baruch is
- `methodology` — How Baruch guides configuration and KB interaction
- `tool_guidance` — How Baruch uses its tools
- `mcp_tool_guidance` — How Baruch uses MCP tools and connected knowledge bases
- `instructions` — Behavioral rules and constraints

Single-tier override: admin KV overrides → hardcoded defaults (no user/mode hierarchy).

### Built-in Tools (14 total)

**BT Servant admin API tools** (8) — calls bt-servant-worker via `ENGINE_API_KEY`:

- Prompt overrides: get, set
- Modes: list, get, create/update, delete
- MCP servers: list, set

**Baruch self-config tools** (4) — reads/writes Baruch's own KV:

- Baruch prompt overrides: get, set
- Baruch MCP servers: get, set

**Memory tools** (2) — internal DO storage:

- Memory: read, update

**MCP tools** (dynamic) — discovered at chat time from configured MCP servers, exposed directly as Claude tools.

### Environments

- **dev**: `baruch-dev` — auto-deployed on PRs to main
- **staging**: `baruch-staging` — deployed when CI passes on main
- **prod**: `baruch` — manual dispatch only

---
> Source: [unfoldingWord/baruch](https://github.com/unfoldingWord/baruch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
