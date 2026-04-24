## claude-code-watchdog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm install                                  # Install dependencies
pnpm build                                    # Build all packages
pnpm test                                     # Run all tests (vitest)
pnpm test -- --run packages/shared            # Run tests for single package
pnpm test -- --run src/db/__tests__/client    # Run single test file
pnpm lint                                     # ESLint + Prettier check

# Database
pnpm --filter @watchdog/shared db:generate    # Generate Prisma client
pnpm --filter @watchdog/shared db:migrate     # Run migrations
pnpm --filter @watchdog/shared db:push        # Push schema to database

# Server
pnpm --filter @watchdog/server start          # Start the server

# Setup (hooks installation)
watchdog-client install                       # Install watchdog hooks
watchdog-client install --path /custom/path   # Install with custom binary path
watchdog-client uninstall                     # Remove watchdog hooks
watchdog-client --help                        # Show usage information
```

## Architecture

Claude Code Watchdog is a security/monitoring layer for Claude Code with three packages:

- **@watchdog/shared** - Prisma schema, database client, and rule engine (used by both client and server)
- **@watchdog/client** - CLI binary (`watchdog-client`) invoked by Claude Code hooks
- **@watchdog/server** - Daemon running SOCKS5 proxy (:1080) and web UI (:8420)

Review [DESIGN.md](DESIGN.md) for how the project is designed to work.

Review [ADRs](docs/adr/) for decisions of record made in development of the project.

### Key Design Decision

The client performs all tool rule matching directly against SQLite (`~/.watchdog/watchdog.db`). Server is only required for web request governance (SOCKS proxy) and management UI. This ensures tool governance works even when server is down.

### Data Flow

1. Claude Code hooks invoke `watchdog-client` with JSON on stdin
2. Client matches tool against rules in SQLite (type: `tool`)
3. Client returns decision to Claude Code via `hookSpecificOutput` with `permissionDecision`
4. Client notifies server via Unix socket (fire-and-forget) for dashboard updates
5. SOCKS proxy uses separate rules (type: `domain`) for web traffic

### Database

- SQLite with WAL mode for concurrent access from client + server
- Schema in `packages/shared/prisma/schema.prisma`
- Four models: `Rule`, `Session`, `ToolUse`, `WebRequest`

### Rule Engine

- **Types**: `tool` (for hooks) vs `domain` (for SOCKS proxy)
- **Matching order**: Manual rules first, then auto; longer patterns win ties
- **Patterns**: Regex - `^Bash$` (exact), `^mcp__.*` (prefix), `.*\.github\.com$` (suffix)
- **`message` field**: Injected as `system_message` for allow rules, used as block `reason` for deny

### Hook Response Decisions

| Rule Match | Return Value                                                                                                                                | Result                              |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `allow`    | `{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "allow", permissionDecisionReason?: "..."}, systemMessage?: "..."}` | Tool executes (bypasses permission) |
| `deny`     | `{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "deny", permissionDecisionReason: "..."}}`                          | Tool blocked                        |
| No rule    | `{}`                                                                                                                                        | User prompted                       |

### Auto-Learning

- 10-minute window: PostToolUse creates auto-allow rule if tool completed successfully
- Client-side only (no server dependency)
- Triggered when user approves tool in Claude Code's permission prompt

### Server Components

| Component     | Address                     | Purpose                                            |
| ------------- | --------------------------- | -------------------------------------------------- |
| Event Monitor | `~/.watchdog/watchdog.sock` | Receives client notifications (fire-and-forget)    |
| SOCKS Proxy   | `:1080`                     | Domain rule enforcement for web traffic            |
| Web UI        | `:8420`                     | Rule management, session viewer, request approvals |

### Environment Variables

| Variable                     | Default                     | Description                 |
| ---------------------------- | --------------------------- | --------------------------- |
| `WATCHDOG_DATABASE_PATH`     | `~/.watchdog/watchdog.db`   | SQLite database path        |
| `WATCHDOG_SOCKET_PATH`       | `~/.watchdog/watchdog.sock` | Unix socket path            |
| `WATCHDOG_SOCKS_PORT`        | `1080`                      | SOCKS5 proxy port           |
| `WATCHDOG_WEB_PORT`          | `8420`                      | Web UI port                 |
| `WATCHDOG_AUTO_LEARN_WINDOW` | `600`                       | Auto-learn window (seconds) |

### Testing

Uses in-memory SQLite databases via test utilities in `packages/shared/src/db/__tests__/testUtils.ts`:

```typescript
const db = await createTestDatabase();
await seedTestData(db.client, { rules: [...] });
// ... test logic
await cleanupTestDatabase(db);
```

**Test Isolation**: Each `createTestDatabase()` call creates a truly unique in-memory database using `pid + counter + randomUUID` to ensure complete isolation between tests, even when running in parallel with `pool: "forks"`. This prevents database state leakage that can cause flaky tests.

**Important**: When modifying code in `@watchdog/shared`, always run `pnpm build` before testing. The server and client packages import via published exports (like `@watchdog/shared/testing`) which point to the compiled `dist/` directory. Without rebuilding, tests will use stale code.

### Web UI

Built with HTMX, do not use vanilla javascript, stick with HTMX

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dinedal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
