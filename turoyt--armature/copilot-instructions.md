## armature

> Opinionated NestJS 11 boilerplate (TypeScript, Prisma, PostgreSQL, Socket.IO).

# Armature — Claude Code guidance

Opinionated NestJS 11 boilerplate (TypeScript, Prisma, PostgreSQL, Socket.IO).

---

## Commands

```bash
npm run start:dev   # Dev server with hot-reload
npm run build       # TypeScript compilation (nest build)
npm run test        # Unit tests (jest)
npm run test:e2e    # End-to-end tests
npm run lint        # ESLint
```

---

## Key conventions

### Errors — always use `ErrorCode`

Never throw raw strings. Use the typed constants in
`src/common/constants/error-constants.ts`:

```ts
throw new NotFoundException(ErrorCode.RESOURCE_NOT_FOUND);
throw new UnauthorizedException(ErrorCode.UNAUTHORIZED);
```

When adding a new code, also add its translation in **both**
`src/common/constants/translations/en.ts` and `fr.ts`. The
`Record<ErrorCode, string>` type enforces this at compile time.

### Serialization — `serialize()` + `@Expose()`

Never return raw Prisma objects. Use `serialize(DtoClass, data)` from
`src/common/utils/serialize.ts`. Only fields decorated with `@Expose()` are
included in the response.

### Guards

| Guard | Decorator | Purpose |
|-------|-----------|---------|
| `JwtAuthGuard` | _(global)_ | JWT on all routes |
| `@Public()` | `src/auth/decorators/public.decorator.ts` | Opt out of JWT guard |
| `RolesGuard` | `@Roles('admin')` | Coarse role check |
| `PermissionsGuard` | `@RequirePermissions('read:resource')` | Fine-grained permissions |
| `OwnershipGuard` | `@UseGuards(OwnershipGuard)` | Resource ownership |

### Optional modules (self-activating)

These modules activate automatically based on env vars — no code change needed:

| Module | Env var required |
|--------|-----------------|
| `QueueModule` | `REDIS_URL` |
| `GoogleAuthModule` | `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET` |
| `PaymentModule` | `STRIPE_SECRET_KEY` |

Pattern: `static register(): DynamicModule` checks `process.env` and returns
an empty module if the dependency is missing.

### Adding a new feature module

1. `nest g module my-feature` — scaffold
2. Create `MyFeatureService` with `PrismaService` + `LoggerService`
3. Create `MyFeatureController` with `@ApiBearerAuth()` + `@ApiTags()`
4. Add to `app.module.ts` imports

---

## WebSocket layer

### Architecture

```
src/websocket/
├── ws.adapter.ts                  Custom IoAdapter — reads CORS from ConfigService
├── websocket.module.ts            Provides & exports Gateway, Registry, Guard, Filter
├── websocket.gateway.ts           Socket.IO gateway — JWT auth, policy-aware emit
├── policies/
│   └── ws-policy.registry.ts      Register event & room access rules
├── guards/
│   └── ws-jwt.guard.ts            Per-event guard (checks client.data.user)
├── filters/
│   └── ws-exception.filter.ts     Catches WsException → emits `error` with i18n message
└── decorators/
    └── ws-current-user.decorator.ts  @WsCurrentUser() — equiv. of @CurrentUser()
```

### Adding real-time events to a module

1. Import `WebsocketModule` in the feature module
2. Inject `WebsocketGateway` in the service, call `void this.ws.emit(event, data).catch(...)`
3. Create `MyWsPolicy implements OnModuleInit` to register access rules via `WsPolicyRegistry`
4. Add `MyWsPolicy` to the feature module's `providers`

### Policy system (row-level security)

```ts
// No policy → broadcast to all authenticated clients
// Policy → evaluated per-socket before delivery

registry.register<MyDto>('my-model:created', (user, data) =>
  user.roles.includes('admin') || data.ownerId === user.id,
);

registry.registerRoomPolicy('my-room', (user) =>
  user.roles.includes('admin'),
);
```

See `src/resource/resource.ws-policy.ts` for the full example.

### WS auth flow

- Token passed in `handshake.auth.token` (preferred) or `Authorization: Bearer` header
- Global `JwtAuthGuard` is HTTP-only — WS auth happens in a `server.use()` middleware registered in `afterInit()`
- Invalid/missing token → handshake rejected before connection is established (client receives `connect_error`)

---

## Claude Code skill

A dedicated Claude Code skill (`/armature`) encodes all the conventions above
as an actionable guide.

### Usage

Type `/armature` in any conversation to load the full guide: error codes,
serialization, guards, WebSocket patterns, optional modules, final checklist.

### Skill sections

| Section | What it covers |
|---------|----------------|
| Error codes | `ErrorCode`, updating both translation files |
| Serialization | `serialize()` + `@Expose()`, paginated DTOs |
| Guards & decorators | Full reference table |
| Controller conventions | `@ApiTags`, `@ApiBearerAuth`, params, pagination |
| Service conventions | Injection, fire-and-forget WS, structured logger |
| New module | Scaffold → service → controller → app.module sequence |
| WebSocket | Emitting, per-event and per-room access policies |
| Optional modules | `DynamicModule` pattern with env var check |
| Env variables | Zod schema, `.env.example` |
| Common mistakes | Bad vs. good practices table |
| Final checklist | To validate before every commit |

---

## MCP server

An MCP (Model Context Protocol) server lives in `mcp/`. It exposes the backend
endpoints as tools that Claude can call directly.

### Setup

No extra install needed — the MCP uses the root project's `node_modules`.
Just make sure you have run `npm install` at the project root.

The MCP connects to the backend via `ARMATURE_BASE_URL` (default:
`http://localhost:3000`). A JWT can be pre-loaded via `ARMATURE_TOKEN`.

### Session persistence

After `auth_login` or `auth_register`, both tokens are written to
`.claude/.armature-session.json` (git-ignored). The session is reloaded on
every MCP startup — no need to log in again after restarting Claude Code.
`auth_logout` deletes the file.

### Available tools

| Tool | Endpoint |
|------|----------|
| `auth_methods` | GET /api/auth/methods |
| `auth_register` | POST /api/auth/register |
| `auth_login` | POST /api/auth/login |
| `auth_refresh` | POST /api/auth/refresh |
| `auth_logout` | POST /api/auth/logout |
| `auth_me` | GET /api/auth/me |
| `resource_list` | GET /api/resources |
| `resource_get` | GET /api/resources/:id |
| `resource_create` | POST /api/resources |
| `resource_update` | PATCH /api/resources/:id |
| `resource_delete` | DELETE /api/resources/:id |
| `health_check` | GET /health |

The MCP is registered in `.claude/settings.json` and starts automatically
with Claude Code. Full documentation: [`docs/mcp.md`](docs/mcp.md).

---

## Environment variables

See `.env.example` and `src/config/env.validation.ts` (Zod schema) for the
full list. The app crashes at startup if required variables are missing.

Required: `DATABASE_URL`, `JWT_SECRET` (≥ 32 chars), `JWT_REFRESH_SECRET`
Optional: `REDIS_URL`, `STRIPE_SECRET_KEY`, `GOOGLE_CLIENT_ID/SECRET`, `CORS_ORIGIN`

---

## Documentation (`docs/` + `mkdocs.yml`)

Every new Markdown file added under `docs/` **must** also be registered in the
`nav:` section of `mkdocs.yml`. Forgetting this step means the page is silently
excluded from the published site.

### Current nav structure

```
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - Architecture: architecture.md
  - Core:
    - Authentication: authentication.md
    - RBAC: rbac.md
    - Logger: logger.md
    - Error Codes & i18n: error-codes.md
    - Optional Modules: optional-modules.md
  - Real-time:
    - WebSocket: websocket.md
  - Integrations:
    - Adding a Social Provider: adding-social-provider.md
    - MCP Server: mcp.md
```

### Rules

- Place new pages in the most relevant section (`Core`, `Real-time`, `Integrations`, …).
- Create a new top-level section only for genuinely distinct feature areas.
- Keep section names short (one or two words).
- After adding a page, verify locally with `mkdocs serve` that it appears in the sidebar.

### Writing style — use Material for MkDocs features

The site uses **Material for MkDocs** with the full feature set enabled. Every
doc page must use these constructs instead of plain prose wherever they add
clarity.

#### Admonitions — mandatory for callouts

Never bury a warning or tip in a plain paragraph. Use the appropriate type:

| Type | When to use |
|------|-------------|
| `!!! danger` | Irreversible actions, security issues, data loss risks |
| `!!! warning` | Behaviour that surprises developers; easy-to-make mistakes |
| `!!! note` | Neutral clarifications that the reader must not miss |
| `!!! info` | Background context or "how it works" asides |
| `!!! tip` | Best-practice shortcuts and recommended patterns |

```markdown
!!! warning "connect_error vs error"
    Auth failures surface as `connect_error`, **not** `error`.
    Always listen to both events on the client.
```

Content must be **indented by 4 spaces** under the opening line.

#### Mermaid diagrams — replace ASCII art

Every ASCII flow (`──►`, `│`, `▼`, `└──`) must be replaced with a fenced
`mermaid` block. Pick the right diagram type:

| Situation | Diagram type |
|-----------|-------------|
| Request/response between actors | `sequenceDiagram` |
| State machine or data flow | `flowchart TD` / `flowchart LR` |
| Database relations | `erDiagram` |

```markdown
​```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: POST /api/auth/login
    S-->>C: { accessToken, refreshToken }
​```
```

#### Tabs — for mutually exclusive alternatives

Use `===` tabs whenever the reader has to choose between options
(dev vs prod, correct vs wrong, framework A vs B):

```markdown
=== "Development"
    ```bash
    npm run start:dev
    ```

=== "Production"
    ```bash
    npm run build && node dist/main.js
    ```
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TuroYT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
