## project-next-hono-sqlite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Nx monorepo combining a Next.js 15 frontend (deployed to Vercel) with a Hono.js backend (deployed to Cloudflare Workers) using Cloudflare D1 (SQLite) database. The project uses pnpm as the package manager.

## Development Commands

### Running the Applications

```bash
# Both frontend and backend in parallel
pnpm dev

# Frontend only (http://localhost:3000)
pnpm dev:frontend

# Backend only (http://localhost:8787)
pnpm dev:backend
```

### Building

```bash
# All applications
pnpm build

# Frontend only
pnpm build:frontend
```

### Testing & Linting

```bash
# Run tests on all projects
pnpm test

# Run linters on all projects
pnpm lint
```

### Database Operations

```bash
# Create D1 database (outputs database_id for wrangler.toml)
pnpm db:create

# Generate migrations from Drizzle schema (after schema changes)
pnpm db:generate

# Run migrations on local database (for development)
pnpm db:migrate:local

# Run migrations on production database
pnpm db:migrate
```

### Deployment

```bash
# Deploy backend to Cloudflare Workers
pnpm deploy:backend
```

## Architecture

### Monorepo Structure

- **apps/frontend**: Next.js 15 application with App Router
- **apps/backend**: Hono.js API on Cloudflare Workers
- **packages/shared-types**: Shared TypeScript types between frontend and backend

### Key Architectural Patterns

**Backend (Hono.js on Cloudflare Workers)**
- Built with **OpenAPIHono** from `@hono/zod-openapi` for automatic API documentation
- Uses **Drizzle ORM** (`drizzle-orm/d1`) for type-safe database operations
- **Zod validation** integrated via `@hono/zod-openapi` for request/response validation
- **Structured logging** with request/response tracking, error logging, and DB query performance monitoring
- Environment bindings are typed via the `Bindings` type (apps/backend/src/types/bindings.ts)
- D1 database accessed through `c.env.DB`, initialized with `createDbClient(c.env.DB)`
- CORS configured for localhost:3000 and localhost:4200
- Database binding name must be "DB" to match code expectations
- Routes organized in `apps/backend/src/routes/` directory
- Drizzle schema defined in `apps/backend/src/db/schema.ts`
- Swagger UI available at http://localhost:8787/docs
- OpenAPI spec available at http://localhost:8787/docs/openapi.json

**Frontend (Next.js)**
- Uses a typed API client class (`ApiClient`) for backend communication (apps/frontend/src/lib/api-client.ts)
- API URL configured via `NEXT_PUBLIC_API_URL` environment variable (defaults to http://localhost:8787)
- All API types imported from `@shared/types` package

**Shared Types Package**
- Located at `packages/shared-types`
- Imported as `@shared/types` in both frontend and backend
- Contains request/response types, database models, and utility types
- Changes here affect both applications

### Database Schema

The D1 database uses Drizzle ORM with Drizzle Kit for migrations:
- **Drizzle schema** in `apps/backend/src/db/schema.ts` (source of truth)
- **Generated migrations** in `apps/backend/drizzle/migrations/` (auto-generated from schema)
- **Database client** in `apps/backend/src/db/client.ts` (typed D1 client)

Current tables:
- `items` table - id, name, description, timestamps
- `fruits` table - id, name, price, quantity, timestamps

When modifying schema:
1. Update `apps/backend/src/db/schema.ts` (Drizzle schema - source of truth)
2. Run `pnpm db:generate` to auto-generate SQL migrations in `drizzle/migrations/`
3. Run `pnpm db:migrate:local` for local dev or `pnpm db:migrate` for production
4. Update shared types in `packages/shared-types/src/index.ts` if needed
5. Update route validators if the schema changes affect validation

### Environment Configuration

**Frontend (.env.local)**
- `NEXT_PUBLIC_API_URL`: Backend API URL (required for production, defaults to localhost:8787)

**Backend (wrangler.toml)**
- Must configure D1 database binding after running `pnpm db:create`:
  ```toml
  [[d1_databases]]
  binding = "DB"
  database_name = "backend-db"
  database_id = "your-database-id-from-create-command"
  ```

**Backend (.dev.vars)** - git-ignored file for local secrets

## Nx-Specific Notes

- Backend targets (serve, deploy, db operations) are defined in `apps/backend/project.json`
- Frontend uses Nx Next.js plugin with auto-inferred targets
- The default base branch is `main` (configured in nx.json)
- Nx caching is enabled for build, lint, and test targets

## Common Workflows

### Adding a New API Endpoint

1. Create Zod schemas inline in your route file using `z` from `@hono/zod-openapi`
2. Define routes using `createRoute()` with OpenAPI metadata
3. Implement route handlers using `app.openapi(route, handler)`
4. Create route file in `apps/backend/src/routes/` and export the app
5. Register routes in `apps/backend/src/index.ts` using `app.route('/', yourRoutes)`
6. Define shared types in `packages/shared-types/src/index.ts`
7. Add corresponding methods to `ApiClient` class in `apps/frontend/src/lib/api-client.ts`

Example pattern (see `apps/backend/src/routes/fruits.ts` for reference)

### Database Schema Changes

1. Update `apps/backend/src/db/schema.ts` (Drizzle schema)
2. Run `pnpm db:generate` to auto-generate SQL migration files
3. Review the generated migration in `apps/backend/drizzle/migrations/`
4. Run `pnpm db:migrate:local` to apply to local database
5. Update TypeScript types in `packages/shared-types/src/index.ts` if needed
6. Update route validators if the schema changes affect validation
7. For production, run `pnpm db:migrate` after deployment

**Note**: Drizzle Kit automatically generates SQL migrations from your TypeScript schema changes. The generated migrations are stored in `apps/backend/drizzle/migrations/` and applied via Wrangler's D1 migration system.

### Testing Backend Locally

The backend runs on http://localhost:8787 with these endpoints:

**Documentation:**
- `GET /docs` - Swagger UI (interactive API documentation)
- `GET /docs/openapi.json` - OpenAPI specification

**Health:**
- `GET /` - Basic health check
- `GET /api/health` - Detailed health status (includes database connection)

**Items (legacy):**
- `GET /api/items` - List all items
- `POST /api/items` - Create a new item
- `DELETE /api/items/:id` - Delete an item

**Fruits (example CRUD with Drizzle + Zod):**
- `GET /api/fruits` - List all fruits
- `GET /api/fruits/:id` - Get single fruit
- `POST /api/fruits` - Create fruit (validated)
- `PUT /api/fruits/:id` - Update fruit (validated)
- `DELETE /api/fruits/:id` - Delete fruit

### Deployment Process

**Frontend**: Connect repository to Vercel with these settings:
- Build Command: `pnpm nx build frontend`
- Output Directory: `dist/apps/frontend/.next`
- Install Command: `pnpm install`
- Environment Variable: `NEXT_PUBLIC_API_URL` (set to Cloudflare Worker URL)

**Backend**: Run `pnpm deploy:backend` (requires Cloudflare Wrangler authentication)

## Observability

The backend includes comprehensive observability features for monitoring, debugging, and performance tracking.

### Structured Logging

All logs are output in JSON format with consistent structure, making them easy to query and analyze in Cloudflare Dashboard.

**Log Levels**: DEBUG, INFO, WARN, ERROR

**Automatic Logging**:
- All HTTP requests/responses with timing and status codes
- Database queries with execution time
- Errors with full stack traces
- User context (userId, requestId) attached to all logs

### Viewing Logs

**Development (Real-time)**:
```bash
# Stream logs from your deployed Worker
wrangler tail

# Stream logs from local dev server
pnpm dev:backend
# Logs appear in terminal
```

**Production (Cloudflare Dashboard)**:
1. Go to Cloudflare Dashboard > Workers & Pages
2. Select your Worker
3. Click "Logs" tab
4. Use the search/filter to find specific requests, errors, or users

### Using Structured Logging in Routes

```typescript
import { createLogger } from '../middleware/logging';
import { createDbLogger } from '../utils/db-logger';

app.openapi(yourRoute, async (c) => {
  const logger = createLogger(c);
  const dbLogger = createDbLogger(logger);

  // Log info messages
  logger.info('Processing request', { customData: 'value' });

  // Log database queries with performance tracking
  const results = await dbLogger.logQuery('SELECT', 'tableName', async () =>
    db.select().from(table)
  );

  // Log warnings
  logger.warn('Something unexpected', { details: 'info' });

  // Log errors with stack traces
  try {
    // ... code
  } catch (error) {
    logger.error('Operation failed', error, { context: 'additional info' });
    throw error;
  }
});
```

### Log Structure

Every log entry includes:
- `timestamp`: ISO 8601 timestamp
- `level`: Log level (DEBUG, INFO, WARN, ERROR)
- `message`: Human-readable message
- `requestId`: Unique ID for the request (useful for tracing)
- `userId`: Authenticated user ID (when available)
- `metadata`: Custom key-value pairs
- `error`: Error details with stack trace (for ERROR level)

Example log output:
```json
{
  "timestamp": "2025-10-19T12:34:56.789Z",
  "level": "INFO",
  "message": "Database query executed",
  "requestId": "1729340096789-abc123def",
  "userId": "user_123abc",
  "metadata": {
    "operation": "SELECT",
    "table": "images",
    "duration": 42
  }
}
```

### Middleware Stack

The following middleware is configured in `apps/backend/src/index.ts`:

1. **errorLogger**: Catches unhandled errors and logs with stack traces
2. **requestLogger**: Logs all incoming requests and outgoing responses with timing

### Performance Monitoring

- **Request Duration**: Automatically tracked for all HTTP requests
- **Database Query Time**: Tracked when using `DbLogger`
- **File Upload Size**: Logged for R2 operations

### Advanced: Cloudflare Workers Analytics

For production deployments, consider enabling:

**Workers Logpush** (Paid plan - $5/month):
- Export logs to external services (Datadog, New Relic, S3, etc.)
- Configure in Cloudflare Dashboard or via wrangler.toml:
  ```toml
  [observability]
  enabled = true
  ```

**Analytics Engine** (for custom metrics):
- Track business metrics beyond standard logs
- Requires separate setup in Worker code

### Best Practices

1. **Use structured logging**: Always pass metadata as objects, not in message strings
   ```typescript
   // Good
   logger.info('User created', { userId: user.id, email: user.email });

   // Avoid
   logger.info(`User ${user.id} created with email ${user.email}`);
   ```

2. **Include context**: Add relevant IDs and metadata to help with debugging

3. **Log at appropriate levels**:
   - DEBUG: Detailed debugging info (rarely used in production)
   - INFO: Normal operations, successful actions
   - WARN: Unexpected but handled situations
   - ERROR: Failures that require attention

4. **Don't log sensitive data**: Avoid logging passwords, tokens, or PII

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duongital)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/duongital)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
