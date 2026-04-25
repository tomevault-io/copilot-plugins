## cmg-dashboard-0-1

> Coding rules for Supabase Edge Functions


# Writing Supabase Edge Functions for CMG Dashboard

You're an expert in writing TypeScript and Deno JavaScript runtime. Generate **high-quality Supabase Edge Functions** for the CMG Dashboard project that adhere to the following best practices. Consider potential use cases within CMG Dashboard such as handling complex webhooks, background processing tasks, or specialized API endpoints requiring proximity to the database.

## General Guidelines

1.  **API Preference:** Try to use Web APIs and Deno's core APIs instead of external dependencies (e.g., use `fetch` instead of Axios, use WebSockets API instead of `node-ws`).
2.  **Shared Code:** If reusing utility methods between Edge Functions, add them to `supabase/functions/_shared` and import using a relative path. Do NOT have cross-dependencies between Edge Functions folders.
3.  **Dependency Imports:** Do NOT use bare specifiers. Prefix external dependencies with `npm:` or `jsr:`. For example, `@supabase/supabase-js` should be `npm:@supabase/supabase-js@2` (always specify a version, e.g., `@2` or `@2.39.8`).
4.  **CDN Imports:** Minimize imports from `@deno.land/x`, `esm.sh`, and `@unpkg.com`. Prefer `npm:` or `jsr:`.
5.  **Node.js Built-ins:** Use Node built-in APIs via the `node:` specifier (e.g., `import process from "node:process"`) when Deno APIs have gaps.
6.  **Serving Requests:** Do NOT use `import { serve } from "https://deno.land/std@0.168.0/http/server.ts"`. Use the built-in `Deno.serve`.
7.  **Environment Variables (Secrets):** The following are pre-populated and should be accessed via `Deno.env.get('VAR_NAME')`:
    *   `SUPABASE_URL`
    *   `SUPABASE_ANON_KEY`
    *   `SUPABASE_SERVICE_ROLE_KEY`
    *   `SUPABASE_DB_URL` (for direct DB connections if ever needed, though generally prefer the JS client)
    For other secrets, use `.env` files and `supabase secrets set`.
8.  **Routing:** A single Edge Function can handle multiple routes. For complex routing, use a library like Hono (`npm:hono@latest`) or Oak (`npm:oak@latest`). Routes must be prefixed with `/function-name`.
9.  **File System:** File write operations are ONLY permitted on the `/tmp` directory.
10. **Background Tasks:** Use `EdgeRuntime.waitUntil(promise)` to run long-running tasks without blocking the response. Do NOT assume `EdgeRuntime` is available in the request/execution context directly; it's a global in the Supabase Edge Runtime.

## Security & Error Handling

1.  **Input Validation:** Always validate incoming request bodies and parameters (e.g., using Zod from `npm:zod@latest` if complex).
2.  **Error Handling:** Implement robust `try...catch` blocks. Return meaningful JSON error responses with appropriate HTTP status codes (e.g., 400 for bad input, 401 for unauthorized, 500 for server errors).
3.  **Authentication/Authorization:**
    *   **User-Invoked Functions:** If an Edge Function is called directly by an authenticated client, verify the JWT. Create a Supabase client with the `SUPABASE_ANON_KEY` and use `await supabase.auth.getUser(jwtFromAuthHeader)`.
    *   **Service-to-Service/Webhooks:** If the function is invoked by a trusted service or webhook, consider API key authentication or a shared secret passed in headers/body, verified by the function.
    *   **Database Access:** Use the `SUPABASE_SERVICE_ROLE_KEY` for database operations requiring admin privileges (see Supabase Client section).

## Supabase Client Usage in Edge Functions

-   **Admin Client (Service Role):** For most database interactions from an Edge Function where RLS might be bypassed or administrative actions are needed.
    ```typescript
    // supabase/functions/some-admin-task/index.ts
    import { createClient } from 'npm:@supabase/supabase-js@2'; // Specify version

    const supabaseAdmin = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    );

    // Usage: const { data, error } = await supabaseAdmin.from('your_table').select('*');
    ```
-   **Anon Client (for Auth checks):** When you need to verify a user's JWT.
    ```typescript
    // supabase/functions/protected-function/index.ts
    import { createClient } from 'npm:@supabase/supabase-js@2';

    const supabaseAnon = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_ANON_KEY')!
    );

    // Inside Deno.serve, get JWT from Authorization header: `const authHeader = req.headers.get('Authorization');`
    // Then: `const { data: { user }, error } = await supabaseAnon.auth.getUser(jwt);`
    ```

## Example Templates

### Basic Function with Error Handling

```typescript
// supabase/functions/greet-user/index.ts
interface ReqPayload { name?: string }

console.info('Greet User function started');

Deno.serve(async (req: Request) => {
  try {
    if (req.method !== 'POST') {
      return new Response(JSON.stringify({ error: 'Method Not Allowed' }), { status: 405, headers: { 'Content-Type': 'application/json' } });
    }

    const payload: ReqPayload = await req.json();
    const name = payload.name || 'there';

    const data = {
      message: `Hello ${name} from the Greet User Edge Function!`,
    };

    return new Response(JSON.stringify(data), {
      headers: { 'Content-Type': 'application/json', Connection: 'keep-alive' },
      status: 200
    });

  } catch (error) {
    console.error('Function error:', error);
    return new Response(JSON.stringify({ error: error.message || 'Internal Server Error' }), {
      status: error instanceof SyntaxError ? 400 : 500, // Example: Bad JSON input vs other errors
      headers: { 'Content-Type': 'application/json' },
    });
  }
});
```

### Using npm Package (e.g., Hono for routing)

```typescript
// supabase/functions/api-router/index.ts
import { Hono } from 'npm:hono@3'; // Specify major version
import { serve } from "https://deno.land/std@0.168.0/http/server.ts"; // Hono often uses this for Deno

const app = new Hono();

app.get('/api-router', (c) => c.json({ message: 'Hello from Hono on Supabase Edge!' }));
app.get('/api-router/items/:id', (c) => {
  const id = c.req.param('id');
  return c.json({ itemId: id, description: `Details for item ${id}` });
});

Deno.serve(app.fetch);
```

Refer to `[coding-rule.mdc](mdc:.cursor/rules/coding-rule.mdc)` for general project standards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/negan947) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
