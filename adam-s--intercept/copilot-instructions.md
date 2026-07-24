## intercept

> Discover a website's internal API via browser traffic interception, create a domain plugin with typed proxy routes. The browser IS the API client — use `browserFetch` first, run elimination to find minimum auth, store in `GenericSessionManager`, verify with `rateLimitedFetch` last.

# Interceptor

Discover a website's internal API via browser traffic interception, create a domain plugin with typed proxy routes. The browser IS the API client — use `browserFetch` first, run elimination to find minimum auth, store in `GenericSessionManager`, verify with `rateLimitedFetch` last.

**After discovery, create a domain plugin** in `domains/<name>/` with routes that serve JSON. Follow `.claude/rules/` for the discovery protocol. See `domains/boardshop/` for working reference code — it has examples of every transport type including XHR pagination, session harvest, embedded JSON, GraphQL, WebSocket, and click-intercept.

## Structure

- `apps/api/` — Hono API server with WebSocket + browser streaming (`@interceptor/api`)
- `apps/web/` — Next.js frontend (`@interceptor/web`)
- `packages/browser/` — Patchright browser automation + transport classifier (`@interceptor/browser`)
- `packages/shared/` — Shared types, validation, debug logging (`@interceptor/shared`)
- `packages/test-server/` — Fake websites on port 4444 for validating discovery (`@interceptor/test-server`)
- `packages/db/` — Drizzle ORM + TimescaleDB (optional)
- `services/python/` — Python worker for IPC bridge
- `domains/boardshop/` — Reference domain with working routes for every transport type

## Quick Commands

```bash
pnpm dev                              # Start all services (API 3001, Web 3000)
pnpm --filter @interceptor/api dev    # API only
pnpm --filter @interceptor/web dev    # Web only
pnpm --filter @interceptor/test-server start  # Test server on 4444
./scripts/ci-local.sh                 # Full CI (run before committing)
```

## Key Architecture

- **Browser WebSocket:** `ws://localhost:3001/browser/stream?profile=<domain>&url=<target>`
- **Traffic capture:** `GET /browser/traffic` (only WS-connected browsers capture traffic)
- **Domain proxy:** `GET /api/<domain>/<path>`
- **Test server sites:** boardshop (embedded JSON), liveboard (WebSocket + protobuf), streamshop (GraphQL + HLS), databoard (gRPC-Web + encoded)
- **Debug logs:** `import { DEBUG } from "@interceptor/shared"` → `/tmp/interceptor-debug/`

## Worktree Agents — DO NOT MODIFY THE MAIN REPO

If you are running in a worktree (`pwd` contains `/tmp/interceptor-worktrees/`):
- **ALL files go in YOUR worktree.** Never write to the original repo.
- Run `pnpm install` once to get dependencies. If it fails, fix the error — do not retry with different flags.
- Before starting the API server: `lsof -ti:PORT | xargs kill 2>/dev/null` to clear the port. Start once. **The server does NOT auto-reload domain file changes.** After editing domain files, `kill -9` the server and restart.
- Edit `register-domains.ts` **in your worktree** to import your domain plugin.
- Test routes through `localhost:PORT/api/domain/route`.
- Connect a browser for traffic capture — curl-only discovery misses transport types.

## Code Ownership

We own ALL the code — packages, apps, domains, scripts, test-server, infrastructure. When something is broken, we fix it at the source with production-quality code. No workarounds, no "document it for later," no dummy values to satisfy a type. If an interface is wrong, fix the interface. If a script wastes calls, rewrite the script. Breaking changes are fine — this is pre-release. Delete dead code, rename freely, change interfaces. Tests must pass.

## Conventions

- Import paths: `@interceptor/[package]`
- TypeScript strict mode; Vitest for tests; Biome for linting
- Frontend API URLs: relative (`/api/...`), not `http://localhost:3001/...`
- Never `git add -A` — stage specific files by name
- Always run `./scripts/ci-local.sh` before committing

---
> Source: [adam-s/intercept](https://github.com/adam-s/intercept) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
