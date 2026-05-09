## sandbox-mcp

> Guidelines for AI agents and human contributors working on this codebase.

# AGENTS.md

Guidelines for AI agents and human contributors working on this codebase.

## Project Overview

This is an MCP server that delegates coding tasks to OpenCode running in Cloudflare Sandboxes. It's built on Cloudflare Workers with Durable Objects, Workflows, Containers, and R2.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     MCP Client (Claude, etc.)                    │
└────────────────────────────┬────────────────────────────────────┘
                             │ MCP Protocol
┌────────────────────────────▼────────────────────────────────────┐
│                   Cloudflare Worker                              │
│  Routes: /mcp, /proxy/*, /session/{id}/                          │
└────────────────────────────┬────────────────────────────────────┘
         │                   │                    │
    ┌────▼────┐       ┌──────▼──────┐      ┌─────▼─────┐
    │ Proxy   │       │ MCP Agent   │      │ Web UI    │
    │ Handler │       │    (DO)     │      │ Proxy     │
    └────┬────┘       └──────┬──────┘      └─────┬─────┘
         │                   │                    │
         │           ┌───────▼───────┐           │
         │           │   Workflow    │           │
         │           └───────┬───────┘           │
         │                   │                   │
    ┌────▼───────────────────▼───────────────────▼────┐
    │                 Cloudflare Sandbox               │
    │   ┌─────────────────────────────────────────┐   │
    │   │           OpenCode Agent                 │   │
    │   │  - Autonomous coding                     │   │
    │   │  - Git operations                        │   │
    │   │  - Web UI on port 4096                   │   │
    │   └─────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────┘
```

### Key Components

| Component | Purpose |
|-----------|---------|
| **Worker** | HTTP routing, MCP endpoint, proxy |
| **MCP Agent (DO)** | Protocol handling, tool implementation |
| **Workflow** | Long-running task orchestration (up to 50min) |
| **Sandbox** | Isolated container running OpenCode |
| **R2** | Session metadata, workspace persistence |

## Tech Stack

- **Runtime**: Cloudflare Workers
- **Language**: TypeScript (strict mode)
- **Framework**: Effect for services, errors, and dependency injection
- **Testing**: Vitest
- **Linting**: Oxlint
- **Formatting**: Oxfmt

## Code Style

### Effect Patterns

Services use Effect's Context/Layer pattern:

```typescript
// Define service interface
interface MyServiceInterface {
  readonly doThing: (id: string) => Effect.Effect<Result, MyError>;
}

// Create Context tag
export class MyService extends Context.Tag("MyService")<
  MyService,
  MyServiceInterface
>() {}

// Create Layer factory
export function makeMyServiceLayer(deps: Deps): Layer.Layer<MyService> {
  return Layer.succeed(MyService, makeMyServiceImpl(deps));
}

// Use in Effect.gen
const program = Effect.gen(function* () {
  const service = yield* MyService;
  return yield* service.doThing("123");
});

// Provide layer and run
Effect.runPromise(Effect.provide(program, layer));
```

### Error Types

Use `Schema.TaggedError` for typed errors:

```typescript
export class MyError extends Schema.TaggedError<MyError>()("MyError", {
  id: Schema.String,
  cause: Schema.String,
}) {
  override get message(): string {
    return `Failed for ${this.id}: ${this.cause}`;
  }
}
```

### Schemas

Use Effect Schema for data validation:

```typescript
const MyModel = Schema.Struct({
  id: Schema.String,
  status: Schema.Literal("active", "stopped"),
  createdAt: Schema.Number,
});
type MyModel = typeof MyModel.Type;
```

For MCP tools, use Zod (required by MCP SDK):

```typescript
export const myToolInputSchema = {
  type: "object",
  properties: {
    id: { type: "string", description: "The ID" },
  },
  required: ["id"],
} as const;
```

## Architecture Decisions

### Cloudflare Bindings

Configured in `wrangler.jsonc`:

| Binding | Type | Purpose |
|---------|------|---------|
| `MCP_AGENT` | Durable Object | MCP protocol handler |
| `Sandbox` | Durable Object | Container instances |
| `SESSIONS_BUCKET` | R2 Bucket | Session/workspace storage |
| `EXECUTE_TASK_WORKFLOW` | Workflow | Task execution |

### Storage Layout

All data is stored in R2 for cross-DO access. Key patterns are defined in `src/storage/keys.ts`:

```
sessions/_index.json           # Global session index
sessions/{sessionId}.json      # Session metadata (flat)
runs/_index.json               # Global runs index
runs/{runId}.json              # Run data (flat)
```

- **Sessions in R2**: Cross-DO access (MCP lib creates separate DO per connection)
- **Runs in R2**: Global index enables cross-session queries without knowing sessionIds

### Zero-Trust Proxy

Real credentials never enter sandbox. Proxy validates JWT, injects real creds:

```
Sandbox → JWT as API key → Proxy → Real credentials → External API
```

**Provider support:** Currently hardcoded to Anthropic. To add other providers, see the [Customizing the Provider](./README.md#customizing-the-provider) section in README. The key files are:
- `src/proxy/services/` - Service configs (target URL, auth injection)
- `src/workflows/helpers/opencode.ts` - OpenCode config and model selection

### Workflows for Long Tasks

Durable Objects evict after 70-140s inactivity. Workflows handle tasks up to 50min with automatic retries.

### Session Restoration

OpenCode state is persisted to R2 and restored when sandboxes restart:

1. **Backup**: After each task, `~/.local/share/opencode/storage` is tarred and saved to R2
2. **Restore**: `ensureSandboxReady()` in `src/workflows/helpers/sandbox.ts` handles idempotent initialization:
   - Configures proxy (for git auth)
   - Restores OpenCode backup from R2
   - Clones repository if needed

This allows sessions to survive sandbox eviction and be resumed from any entry point (MCP tool or web UI).

## File Organization

```
src/
├── index.ts              # Worker entry, HTTP routing
├── agent/
│   ├── mcp-agent.ts      # MCP Durable Object
│   └── tools.ts          # Tool schemas and formatters
├── workflows/
│   ├── execute-task.ts   # Main workflow
│   └── helpers/
│       ├── backup.ts     # Session backup/restore to R2
│       ├── opencode.ts   # OpenCode SDK integration
│       ├── run.ts        # Run record management
│       └── sandbox.ts    # Sandbox initialization
├── proxy/
│   ├── handler.ts        # Proxy routing
│   ├── token.ts          # JWT creation/verification
│   └── services/         # Per-service proxy logic
├── services/
│   ├── session.ts        # R2 session storage
│   └── run.ts            # R2 run storage
└── models/
    ├── session.ts        # SessionMetadata schema
    ├── run.ts            # RunRecord schema
    └── errors.ts         # All error types
```

## Common Tasks

### Adding a New MCP Tool

1. Add schema in `src/agent/tools.ts`
2. Register in `src/agent/mcp-agent.ts` via `this.server.registerTool()`
3. Add tests

### Adding a New Error Type

1. Add error class in `src/models/errors.ts` using `Schema.TaggedError`
2. Add type guard if needed (e.g., `isMyError`)
3. Handle in `formatDomainError` in `mcp-agent.ts`

### Modifying Storage

Sessions are in R2 via `SessionStorage` service. Runs are in R2 via `RunStorage` service.

To add a field to SessionMetadata:
1. Update schema in `src/models/session.ts`
2. Update `SessionIndexEntry` in `src/services/session.ts` if needed for listing
3. Update tests

### Adding a Workflow Step

1. Add helper function in `src/workflows/helpers/`
2. Add step in `src/workflows/execute-task.ts`
3. Handle errors appropriately (Workflow retries on throw)

## Testing

```bash
npm run test              # Run all tests
npm run test -- --watch   # Watch mode
npm run test -- path/to/file.test.ts  # Single file
```

Tests use mock layers for Effect services. Example:

```typescript
const createMockBucket = () => {
  const store = new Map<string, string>();
  return {
    get: async (key) => store.get(key) ? { json: async () => JSON.parse(store.get(key)!) } : null,
    put: async (key, value) => { store.set(key, value); },
    // ...
  } as unknown as R2Bucket;
};
```

## Validation Commands

Before committing:

```bash
npm run check  # Runs: typecheck, format:check, lint, knip, test
```

Pre-commit hooks run lint and format checks automatically.

## Gotchas

1. **MCP creates separate DO per connection**: That's why sessions are in R2, not DO storage
2. **Workflows have 50min timeout**: Configured in execute-task.ts
3. **Proxy JWT is short-lived (2h)**: Created per-run in mcp-agent.ts
4. **R2 index uses optimistic locking**: Retry on concurrent modification
5. **Effect services need Layer**: Use `Effect.provide(effect, layer)` to run
6. **Zod for MCP, Effect Schema for models**: Different validation libs for different purposes

---
> Source: [ghostwriternr/sandbox-mcp](https://github.com/ghostwriternr/sandbox-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
