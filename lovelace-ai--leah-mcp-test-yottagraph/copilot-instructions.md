## leah-mcp-test-yottagraph

> Nitro server-side API routes and utilities in server/


# Nitro Server Routes

The `server/` directory contains Nuxt's Nitro server layer. These routes deploy
with the app to Vercel -- they are NOT a separate service. They handle
server-side concerns like KV storage, database access, and image proxying
that can't run in the browser.

## Directory Layout

```
server/
├── api/
│   ├── kv/                  # KV (Upstash Redis) CRUD — read, write, delete, documents, status
│   └── avatar/[url].ts      # Avatar image proxy
└── utils/
    ├── redis.ts              # Upstash Redis client init (from Vercel KV env vars)
    └── cookies.ts            # Cookie handling (@hapi/iron)
```

## Adding Routes

Follow Nitro file-based routing. The filename determines the HTTP method and
path:

```
server/api/my-resource.get.ts      → GET  /api/my-resource
server/api/my-resource.post.ts     → POST /api/my-resource
server/api/my-resource/[id].get.ts → GET  /api/my-resource/:id
```

Route handler pattern:

```typescript
export default defineEventHandler(async (event) => {
  const params = getQuery(event);        // query string
  const body = await readBody(event);    // POST body
  const id = getRouterParam(event, 'id'); // path params

  // ... implementation ...
  return { result: 'data' };
});
```

## KV Storage (Upstash Redis)

`server/utils/redis.ts` initializes the Upstash Redis client from env vars
that Vercel auto-injects when a KV store is connected:

- `KV_REST_API_URL` — Redis REST API endpoint
- `KV_REST_API_TOKEN` — Auth token

```typescript
import { getRedis, toRedisKey } from '~/server/utils/redis';

const redis = getRedis();
if (redis) {
  await redis.hset(toRedisKey('/users/abc/settings'), { theme: 'dark' });
  const theme = await redis.hget(toRedisKey('/users/abc/settings'), 'theme');
}
```

Returns `null` if KV is not configured (env vars missing). Always check.

## Supabase (PostgreSQL)

If Supabase is connected to the project, Vercel auto-injects these env vars:

- `NUXT_PUBLIC_SUPABASE_URL` — Supabase project URL
- `NUXT_PUBLIC_SUPABASE_ANON_KEY` — Public anon key (safe for client-side)
- `SUPABASE_SERVICE_ROLE_KEY` — Server-only service role key (never expose to client)
- `SUPABASE_DB_URL` — Direct Postgres connection string

Use `@supabase/supabase-js` for the Supabase client:

```typescript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NUXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!,  // server routes only
);

const { data } = await supabase.from('my_table').select('*');
```

For client-side access, use the anon key instead of the service role key.

## Key Differences from Client-Side Code

- Server routes run on the server (Node.js), not in the browser
- They have access to Redis, Supabase, secrets, and server-only APIs
- They do NOT have access to Vue composables, Vuetify, or any client-side code
- Use `defineEventHandler`, not Vue component patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
