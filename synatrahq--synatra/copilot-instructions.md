## synatra

> ├── app.ts              # Hono app setup

## Package Structure

```
src/
├── app.ts              # Hono app setup
├── auth.ts             # better-auth configuration
├── config.ts           # Environment config
├── temporal.ts         # Temporal client
├── middleware/         # Hono middlewares
├── routes/             # API endpoints
└── util/               # Shared utilities
```

## Routes Structure

### Directory Layout

```
routes/
├── {resource}/           # Resource endpoints (agents, triggers, etc.)
│   ├── index.ts          # Aggregates and exports all routes
│   ├── list.ts           # GET /
│   ├── get.ts            # GET /:id
│   ├── create.ts         # POST /
│   ├── update.ts         # PATCH /:id
│   ├── delete.ts         # DELETE /:id
│   └── {nested}/         # Nested resources (releases, etc.)
│       ├── index.ts
│       └── list.ts       # GET /:id/{nested}
├── webhook.ts            # Standalone: POST /webhook/:orgSlug/:envSlug/:triggerSlug
├── trigger-run.ts        # Standalone: POST /trigger/:orgSlug/:envSlug/:triggerSlug/:version/run
└── app-webhook.ts        # Standalone: POST /apps/:appId/webhook
```

### File Rules

- 1 endpoint = 1 file
- File name = action name (`list.ts`, `create.ts`, `toggle.ts`)
- `index.ts` is the only exception (aggregates routes)

### Endpoint File Pattern

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { createResource, CreateResourceSchema } from "@synatra/core"
import { requirePermission } from "../../middleware/principal"

export const create = new Hono().post(
  "/",
  requirePermission("resource", "create"),
  zValidator("json", CreateResourceSchema),
  async (c) => {
    const body = c.req.valid("json")
    const result = await createResource(body)
    return c.json(result, 201)
  },
)
```

### Zod Schema Rules

- Import schemas from `@synatra/core` (e.g., `CreateAgentSchema`, `UpdateThreadSchema`)
- Use `zValidator("json", Schema)` for request body
- Use `zValidator("query", Schema)` for query params
- Access validated data via `c.req.valid("json")` or `c.req.valid("query")`
- Only define file-local schemas for server-specific validation (query params, headers)

### index.ts Pattern

```typescript
import { Hono } from "hono"
import { list } from "./list"
import { get } from "./get"
import { create } from "./create"
import { update } from "./update"
import { del } from "./delete"

export const resources = new Hono()
  .route("/", list)
  .route("/", get)
  .route("/", create)
  .route("/", update)
  .route("/", del)
```

## API Patterns

### HTTP Methods & Paths

- `GET /` - list resources
- `GET /:id` - get single resource
- `POST /` - create resource (201)
- `PATCH /:id` - update resource
- `DELETE /:id` - delete resource

### Nested Resources

- `GET /:id/releases` - list nested
- `POST /:id/releases` - create nested

### Responses

- Create: status 201
- Async operations: status 202
- Delete: `{ id, deleted: true }`

### Error Handling

- Use `createError(name, data)` from @synatra/util/error
- Errors are caught by global error handler

### Authorization Middleware

```typescript
requirePermission("resource", "action") // RBAC permission check
requireAuth // Authentication required
requireOrganization // Organization context required
```

### Principal Scope

```typescript
principal.orgId() // Current organization ID
principal.userId() // Current user ID
```

### Organization Scoping

Core `get{Entity}ById`/`find{Entity}ById` scope by organization automatically. No manual checks needed.

## Business Logic

- All business logic lives in `@synatra/core`
- Server routes are thin HTTP handlers only
- Import both functions and schemas from core
- Never duplicate validation or business rules in server

## Utilities

### bearer-auth.ts

- `extractBearerToken(header)` - Extract token from Authorization header
- `verifySecret(provided, expected)` - Timing-safe secret comparison

### signed-state.ts

- `signState<T>(data)` - Sign data with HMAC-SHA256
- `verifyState<T>(state, schema)` - Verify and parse with Zod schema

### validate-payload.ts

- `validatePayload(payload, schema)` - AJV JSON Schema validation

## Tech Stack

- Hono (with RPC client generation)
- better-auth (authentication)
- Temporal (workflow orchestration)
- Zod (validation)

---
> Source: [synatrahq/synatra](https://github.com/synatrahq/synatra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
