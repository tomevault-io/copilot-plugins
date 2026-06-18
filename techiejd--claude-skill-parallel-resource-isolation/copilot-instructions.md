## claude-skill-parallel-resource-isolation

> Use when about to start a dev server, run e2e tests, or connect to a database — proactively detects resource conflicts from other running processes and silently adapts to avoid collisions


# Parallel Resource Isolation

## Overview

Multiple Claude instances working on the same project will collide on ports, databases, and browser instances. This skill teaches proactive detection and silent adaptation — check what's in use, pick something else, report what you chose.

**Core principle:** Observe system state, adapt silently, report in terminal. No coordination files, no lock files, no shared state.

## When to Use

Before any of these actions, run the detection checks:
- Starting a dev server (`next dev`, `npm run dev`)
- Running Playwright e2e tests
- Running tests that write to or assert against the database

## Port Isolation (Dev Server)

Before starting a dev server:

```bash
lsof -i :3000 -t 2>/dev/null
```

If occupied, scan upward (3001, 3002, ...) until a free port is found. Start with:

```bash
PORT=<free_port> next dev
```

**Playwright with `webServer` config:**
- Override port via env var, pass same port as `baseURL`
- If env-based port override isn't supported, flag the user — do NOT modify config files

**Playwright without `webServer` (external server):**
- Start dev server on the free port
- Run Playwright with overridden baseURL:

```bash
BASE_URL=http://localhost:<free_port> npx playwright test
```

**Hardcoded `baseURL` in Playwright config (common case):**
Most configs hardcode `baseURL: 'http://localhost:3000'` instead of reading from env. If so, `BASE_URL=...` won't work. Flag the user with this exact fix:

> Your `playwright.config.ts` has a hardcoded `baseURL`. Please update it to:
> `baseURL: process.env.BASE_URL || 'http://localhost:3000'`
> This lets me override the port via env var without modifying config files.

## Database Isolation (Supabase)

Before running tests that write to or assert against the DB:

1. Determine if isolation is needed:
   - **Reading only** → share the DB, skip isolation
   - **Writing / mutating / asserting on DB state** → isolate
2. Generate a timestamp-based schema prefix: `parallel_<epoch>_` (e.g., `parallel_1741234567_`)
3. Create the isolated schema and run migrations:

```bash
psql "$DATABASE_URL" -c "CREATE SCHEMA parallel_1741234567_;"
PGOPTIONS="--search_path=parallel_1741234567_" npx supabase db push
```

4. Point the app at the isolated schema via env var:

```bash
SUPABASE_SCHEMA=parallel_1741234567_ npx playwright test
```

5. **After tests complete:** drop the schema:

```bash
psql "$DATABASE_URL" -c "DROP SCHEMA parallel_1741234567_ CASCADE;"
```

## Playwright Browser Isolation

Before running Playwright tests:

1. Check for running Playwright browser processes that could conflict
2. Isolate browser state to avoid lock contention — use either approach:

```bash
TMPDIR=/tmp/playwright_<epoch> npx playwright test
```

Or more targeted — isolate only the browser path:

```bash
PLAYWRIGHT_BROWSERS_PATH=/tmp/pw_browsers_<epoch> npx playwright install chromium
PLAYWRIGHT_BROWSERS_PATH=/tmp/pw_browsers_<epoch> npx playwright test
```

3. Ensure the Playwright instance targets the same port assigned in Port Isolation above

## Reporting

After adapting, report once and move on:

```
⚡ Parallel isolation:
  Dev server: port 3005 (3000 was in use)
  DB schema: parallel_1741234567_
  Playwright: /tmp/playwright_1741234567
```

Only report resources that were actually adapted. If nothing conflicted, say nothing.

## Quick Reference

| Resource | Detection | Adaptation | Cleanup |
|----------|-----------|------------|---------|
| Dev server port | `lsof -i :<port>` | `PORT=<next_free>` | None needed |
| Supabase DB | Check if tests write/assert | `parallel_<epoch>_` schema | Drop schema after tests |
| Playwright browser | Check for lock contention | `TMPDIR` or `PLAYWRIGHT_BROWSERS_PATH` | Temp dir auto-cleaned |

## Common Mistakes

### Killing the other Claude's process
- **Problem:** Port 3000 is in use, so you kill the process to free it
- **Fix:** Never kill processes you didn't start. Pick a different port.

### Modifying config files for port overrides
- **Problem:** Editing `playwright.config.ts` or `next.config.js` creates git conflicts
- **Fix:** Use environment variables. If env vars don't work, flag the user.

### Forgetting to pass the adapted port to Playwright
- **Problem:** Dev server on 3005, Playwright still hits 3000
- **Fix:** Always coordinate — the port assigned to the dev server must be passed as `BASE_URL` to Playwright.

### Isolating the DB when not needed
- **Problem:** Overhead of schema creation for read-only tests
- **Fix:** Only isolate when tests write or assert on DB state.

## Red Flags

**Never:**
- Kill another process to free a resource
- Modify config files to change ports
- Ignore port conflicts and hope for the best
- Skip DB cleanup after tests

**Always:**
- Check before starting
- Adapt silently via env vars
- Report what you chose
- Clean up DB schemas after tests

---
> Source: [techiejd/claude-skill-parallel-resource-isolation](https://github.com/techiejd/claude-skill-parallel-resource-isolation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
