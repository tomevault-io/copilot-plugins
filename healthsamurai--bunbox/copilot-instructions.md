## bunbox

> Use Bun instead of Node.js, npm, pnpm, or vite.


# Bunbox - FHIR R4 Server

## Quick Start

```bash
# Start PostgreSQL
docker-compose up -d

# Install dependencies
bun install

# Initialize database schema
psql postgres://postgres:postgres@localhost:54321/bunbox < schema.sql

# Run server
bun src/index.ts

# Run tests
bun test
```

## Project Structure

```
src/
  index.ts              # HTTP server entry point (Bun.serve)
  db.ts                 # PostgreSQL connection, UUIDv7, table helpers
  fhir/
    crud.ts             # FHIR CRUD operations (create, read, update, delete, search)
    history.ts          # Version history operations (vread, instance-history)
    validate-patient.ts # Patient validation (original)
    validate-patient-fast.ts # Patient validation (optimized, use this one)

tests/
  fhir-crud.test.ts     # CRUD acceptance tests
  validate-patient*.ts  # Validation tests

perf/
  create.k6.js          # k6 load test for creates
  read.k6.js            # k6 load test for reads
  create-validation.k6.js # Validation overhead benchmark
  validate-patient.bench.ts # Pure validation function benchmark
  reports/              # Benchmark results
```

## Database

- PostgreSQL with JSONB storage
- Multi-tenant: `tenant_id` column on all tables
- Patient-related resources have `patient_id` for efficient queries
- History tables (`*_history`) for versioning
- UUIDv7 for IDs (time-sortable)

## API Endpoints

```
GET  /metadata              # CapabilityStatement
GET  /:type                 # Search
GET  /:type/:id             # Read
GET  /:type/:id/_history    # Instance history
GET  /:type/:id/_history/:vid # Version read
POST /:type                 # Create
PUT  /:type/:id             # Update
DELETE /:type/:id           # Delete

POST /$toggle-validation    # Toggle validation {"enabled": true/false}
GET  /$validation-status    # Check validation status
```

## Performance

- Create: ~12K req/s (with validation)
- Read: ~30K req/s
- Validation: ~2M validations/sec (optimized)
- Validation overhead: ~3.7%

## Bun Conventions

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`.
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`.
- `Bun.redis` for Redis. Don't use `ioredis`.
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`.
- `WebSocket` is built-in. Don't use `ws`.
- Prefer `Bun.file` over `node:fs`'s readFile/writeFile
- Bun.$`ls` instead of execa.

## Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

## Frontend

Use HTML imports with `Bun.serve()`. Don't use `vite`. HTML imports fully support React, CSS, Tailwind.

Server:

```ts#index.ts
import index from "./index.html"

Bun.serve({
  routes: {
    "/": index,
    "/api/users/:id": {
      GET: (req) => {
        return new Response(JSON.stringify({ id: req.params.id }));
      },
    },
  },
  // optional websocket support
  websocket: {
    open: (ws) => {
      ws.send("Hello, world!");
    },
    message: (ws, message) => {
      ws.send(message);
    },
    close: (ws) => {
      // handle close
    }
  },
  development: {
    hmr: true,
    console: true,
  }
})
```

HTML files can import .tsx, .jsx or .js files directly and Bun's bundler will transpile & bundle automatically. `<link>` tags can point to stylesheets and Bun's CSS bundler will bundle.

```html#index.html
<html>
  <body>
    <h1>Hello, world!</h1>
    <script type="module" src="./frontend.tsx"></script>
  </body>
</html>
```

With the following `frontend.tsx`:

```tsx#frontend.tsx
import React from "react";
import { createRoot } from "react-dom/client";

// import .css files directly and it works
import './index.css';

const root = createRoot(document.body);

export default function Frontend() {
  return <h1>Hello, world!</h1>;
}

root.render(<Frontend />);
```

Then, run index.ts

```sh
bun --hot ./index.ts
```

For more information, read the Bun API docs in `node_modules/bun-types/docs/**.mdx`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HealthSamurai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
