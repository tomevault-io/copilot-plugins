## bot-ville

> - **Never import `@repo/core` from `@repo/shared`** — no `import ... from "@repo/core"` in any `packages/shared/src/*.ts` file; shared is a leaf package with zero internal deps

# Anti-Patterns

## Architecture Violations
- **Never import `@repo/core` from `@repo/shared`** — no `import ... from "@repo/core"` in any `packages/shared/src/*.ts` file; shared is a leaf package with zero internal deps
- **Never import `@repo/memory` from `@repo/shared` or `@repo/core`** — memory is an independent package
- **Never put I/O in `@repo/core`** — engine (`packages/core/src/engine.ts`) is pure logic; all I/O goes through the `PersistenceAdapter` interface (`packages/core/src/persistence.ts`); backend provides `PrismaAdapter` in `apps/backend/src/persistence/prisma-adapter.ts`
- **Never call `/api/*` routes from the web app** — `apps/web/` is read-only; use only `publicApi` from `apps/web/app/lib/api.ts` which calls `/public/*` endpoints
- **Never bypass `enforcePolicy()`** — import from `@repo/core/policy-engine`; all role-gated actions in `FarmEngine` must check policy before executing; throws `PolicyViolationError` on violation
- **Never skip status transition validation** — use `isValidTransition(from, to)` from `@repo/shared/statuses` before calling `engine.updateTaskStatus()`

## Data Handling
- **Never store raw JSON objects in SQLite** — Prisma stores JSON as strings; use `PrismaAdapter` helper methods (`toWorkOrder()`, `toTask()`, `toArtifact()`, `toDecision()`, `toEventLog()`) for deserialization and `JSON.stringify()` when writing `context`, `deps`, or `payload` fields
- **Never hardcode `OPENAI_API_KEY`** or any secret — read from `process.env`; `OpenAIEmbedder` accepts key via constructor options or env
- **Never log event payloads containing secrets** — events are persisted to `EventLog` table and broadcast via WebSocket to all connected clients
- **Never mutate canonical artifacts directly** — only `GRAIN_ELEVATOR` role via `MergeEngine.merge()` (in `packages/core/src/merge/merge-engine.ts`) can set `canonical: true`

## Code Style
- **Never use CommonJS** (`require`/`module.exports`) — all packages are ESM (`"type": "module"`); use `import`/`export` only
- **Never omit `.js` extension in relative ESM imports** — e.g., `import { FarmEngine } from "./engine.js"` not `"./engine"`
- **Never define prop types in separate `types` files** — use inline object types on React components
- **Never use `.prettierrc`** — project uses Prettier defaults only
- **Never create non-flat ESLint configs** — all configs must be flat format (`eslint.config.mjs`), extending from `packages/eslint-config/base.js`
- **Never omit `"use client"` on shared UI components** — all `@repo/ui/src/*.tsx` components need the directive as their first line
- **Never use `@base-ui/react` primitives directly in app code** — import from `@repo/ui/*` instead (e.g., `@repo/ui/button`, not `@base-ui/react/button`)
- **Never use `__dirname` or `__filename`** — use `import.meta.url` with `fileURLToPath()` for path resolution in ESM

## Testing
- **Never depend on SQLite in unit tests** — use `InMemoryAdapter` from `packages/core/src/testing/in-memory-adapter.ts` (`import { InMemoryAdapter } from "../src/testing/in-memory-adapter.js"`)
- **Never skip Zod validation on API boundaries** — all route handlers must parse request bodies with schema `.parse()` (e.g., `CreateWorkOrderRequest.parse(request.body)`, `AssignTaskRequest.parse(request.body)` from `@repo/shared/api`)
- **Never forget `registerAllRoles()` in test setup** — call it in `beforeEach` before creating `FarmEngine` instances

## Agent Management
- **Never spawn agents without a role assignment** — every `AgentSession` requires a `roleId` from `RoleIdEnum`; `AgentSpawner.spawn()` enforces this via `SpawnOptions`
- **Never skip worktree cleanup** — call `WorktreeManager.remove()` or `WorktreeManager.removeWithBranch()` when agent sessions end; `AgentSpawner.prune()` cleans up stale sessions
- **Never kill agents without updating session status** — use `AgentSpawner.kill(sessionId)` which handles both process termination and status update

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmsaucier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
