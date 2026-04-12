## sftw-tempo

> Coding rules for Supabase Edge Functions


# Writing Supabase Edge Functions (IT CARGO Project)

You're an expert in writing TypeScript and Deno JavaScript runtime. Generate **high-quality Supabase Edge Functions** for the IT CARGO project, adhering to the following best practices:

## Guidelines

1.  **API Preference**: Try to use Web APIs (like `fetch`, `URL`, `Response`, `Request`) and Deno's core APIs instead of external dependencies where feasible (e.g., use `fetch` instead of Axios, use native WebSocket API if needed).
2.  **Shared Code**: If you are reusing utility methods (e.g., data transformation, specific formatting for IT CARGO) between Edge Functions, add them to `supabase/functions/_shared/` directory (e.g., `supabase/functions/_shared/itc-utils.ts`). Import using a relative path (e.g., `import { helper } from "../_shared/itc-utils.ts"`). Do NOT have direct cross-dependencies between different Edge Function directories (e.g., `import ... from "../my-other-function/somefile.ts"`).
3.  **Dependency Imports**: Do NOT use bare specifiers when importing dependencies. If you need to use an external dependency, it MUST be prefixed with either `npm:` or `jsr:`. For example, `@supabase/supabase-js` should be imported as `npm:@supabase/supabase-js`.
4.  **Dependency Versioning**: For external imports (npm, jsr), always define a specific version. For example, `npm:@supabase/supabase-js@2.39.0`.
5.  **Preferred CDNs/Registries**: For external dependencies, importing via `npm:` and `jsr:` is preferred. Minimize the use of imports from `deno.land/x`, `esm.sh`, and `unpkg.com`. If a package is available on npm, prefer the `npm:` specifier.
6.  **Node.js Built-in APIs**: You can use Node.js built-in APIs. Import them using the `node:` specifier (e.g., `import process from "node:process"`). Use Node APIs when Deno APIs have gaps for specific needs.
7.  **Serving Requests**: Do NOT use `import { serve } from "https://deno.land/std@0.168.0/http/server.ts"` or older std library versions. Instead, use the built-in `Deno.serve` for handling HTTP requests.
8.  **Pre-populated Secrets**: The following environment variables are automatically available in both local and hosted Supabase Edge Function environments. You do not need to (and should not) set them manually in `.env` files for the function itself:
    *   `SUPABASE_URL`
    *   `SUPABASE_ANON_KEY`
    *   `SUPABASE_SERVICE_ROLE_KEY`
    *   `SUPABASE_DB_URL` (direct database connection string, use with caution)
9.  **Custom Secrets**: To set other environment variables (e.g., API keys for third-party services IT CARGO might integrate with), use a `.env` file locally (`supabase/functions/.env`) and deploy them using `supabase secrets set --env-file path/to/your/project-level/.env.local` (or similar, check Supabase CLI docs for exact command for secrets).
10. **Routing**: A single Edge Function can handle multiple routes. For complex routing within a function, it is recommended to use a lightweight router like Hono (`npm:hono`) or Oak (`deno.land/x/oak` if necessary, but prefer npm/jsr if Hono fits). Each route served by the function will be accessed under the base path `/functions/v1/your-function-name/your-route`.
11. **File Operations**: File write operations are ONLY permitted in the `/tmp` directory. Use Deno or Node File APIs for this.
12. **Background Tasks**: Use `EdgeRuntime.waitUntil(promise)` to run long-running tasks (e.g., sending an email after response) without blocking the response to the client. Do NOT assume `EdgeRuntime` is available in the global scope directly; it might be part of a specific framework context or need to be accessed carefully. Check Supabase documentation for the most current way to access this.
13. **Error Handling**: Implement robust error handling. Return appropriate HTTP status codes (e.g., 200, 201, 400, 401, 403, 404, 500) and a consistent JSON error message body (e.g., `{ "error": "Descriptive message", "details": "optional_details" }`). Log errors effectively (e.g., `console.error`) for debugging in Supabase Function logs.
14. **Security**: 
    *   When interacting with Supabase services, use the `SUPABASE_SERVICE_ROLE_KEY` *only when absolutely necessary* for operations that genuinely require bypassing RLS (e.g., an admin task, a trusted backend process). 
    *   For user-driven actions, if the user's JWT is passed to the Edge Function (e.g., in the `Authorization` header), create a Supabase client instance initialized with this JWT. This ensures that database operations respect the user's RLS policies. Example:
        ```typescript
        // Inside Deno.serve, after getting req
        const authHeader = req.headers.get('Authorization');
        if (!authHeader) return new Response("Unauthorized", { status: 401 });
        const supabaseClient = createClient(Deno.env.get('SUPABASE_URL')!, Deno.env.get('SUPABASE_ANON_KEY')!, {
          global: { headers: { Authorization: authHeader } }
        });
        // Now use supabaseClient for user-context queries
        ```
15. **Request Validation**: Validate incoming request bodies and parameters (e.g., query parameters, JSON payloads). For IT CARGO, this is crucial for functions handling pre-alerts or marketplace data. Consider using a library like Zod (`npm:zod`) for complex validation, or simple type checks and assertions for basic functions. Ensure type definitions for payloads are clear (interfaces/types).
16. **Idempotency**: For functions that perform write operations (e.g., creating a resource), consider how to handle retries or duplicate requests if applicable (e.g., by checking if a resource with a given identifier already exists).

## Example Templates for IT CARGO

### Simple Function (e.g., for a quick utility)

```typescript
// supabase/functions/itc-simple-logger/index.ts
console.log('IT CARGO Simple Logger function loaded');

Deno.serve(async (req: Request) => {
  try {
    const payload = await req.json();
    console.log('Received payload:', payload);

    // Example: Echo back a modified message
    const responseData = {
      received: true,
      message: `IT CARGO processed: ${payload.message || 'No message'}`,
      timestamp: new Date().toISOString(),
    };

    return new Response(JSON.stringify(responseData), {
      headers: { 'Content-Type': 'application/json', 'Connection': 'keep-alive' },
      status: 200,
    });
  } catch (error) {
    console.error('Error in itc-simple-logger:', error);
    return new Response(JSON.stringify({ error: 'Failed to process request', details: error.message }), {
      headers: { 'Content-Type': 'application/json' },
      status: 400, // Or 500 if it's an internal server error
    });
  }
});
```

### Function Using Supabase Client (User Context for RLS)

```typescript
// supabase/functions/itc-submit-prealert/index.ts
import { createClient, SupabaseClient } from 'npm:@supabase/supabase-js@2.39.0';

interface PreAlertPayload {
  tracking_identifier_from_supplier: string;
  // ... other fields for shipment_pre_alerts table
  company_id: string; // Assuming this is known or derived
}

console.log('IT CARGO Submit Pre-alert function loaded');

Deno.serve(async (req: Request) => {
  try {
    const authHeader = req.headers.get('Authorization');
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return new Response(JSON.stringify({ error: 'Missing or invalid authorization token' }), { status: 401, headers: { 'Content-Type': 'application/json' } });
    }

    const supabaseUrl = Deno.env.get('SUPABASE_URL');
    const supabaseAnonKey = Deno.env.get('SUPABASE_ANON_KEY');

    if (!supabaseUrl || !supabaseAnonKey) {
      console.error('Supabase URL or Anon Key not set in environment.');
      return new Response(JSON.stringify({ error: 'Server configuration error' }), { status: 500, headers: { 'Content-Type': 'application/json' } });
    }

    const supabaseClient: SupabaseClient = createClient(supabaseUrl, supabaseAnonKey, {
      global: { headers: { Authorization: authHeader } },
    });

    const payload: PreAlertPayload = await req.json();

    // Add validation for payload here (e.g., using Zod)
    if (!payload.tracking_identifier_from_supplier || !payload.company_id) {
        return new Response(JSON.stringify({ error: 'Missing required fields: tracking_identifier_from_supplier, company_id' }), { status: 400, headers: { 'Content-Type': 'application/json' } });
    }

    const { data, error } = await supabaseClient
      .from('shipment_pre_alerts') // RLS will apply based on user's company_id
      .insert({
        tracking_identifier_from_supplier: payload.tracking_identifier_from_supplier,
        company_id: payload.company_id,
        user_id: (await supabaseClient.auth.getUser()).data.user?.id, // Get user_id from token
        // ... other fields from payload
        status: 'SUBMITTED_BY_CLIENT', // Initial status
      })
      .select();

    if (error) {
      console.error('Supabase insert error:', error);
      return new Response(JSON.stringify({ error: 'Failed to submit pre-alert', details: error.message }), {
        status: error.code === '23505' ? 409 : 500, // Example: conflict or other server error
        headers: { 'Content-Type': 'application/json' },
      });
    }

    // Potentially trigger a notification via EdgeRuntime.waitUntil() here
    // const emailPromise = sendConfirmationEmail(data[0]);
    // if (typeof EdgeRuntime !== 'undefined' && EdgeRuntime.waitUntil) {
    //   EdgeRuntime.waitUntil(emailPromise);
    // } else {
    //   await emailPromise; // Fallback or handle differently if EdgeRuntime not available
    // }

    return new Response(JSON.stringify({ success: true, preAlert: data ? data[0] : null }), {
      headers: { 'Content-Type': 'application/json' },
      status: 201, // Created
    });

  } catch (error) {
    console.error('General error in itc-submit-prealert:', error);
    return new Response(JSON.stringify({ error: 'An unexpected error occurred', details: error.message }), {
      headers: { 'Content-Type': 'application/json' },
      status: error instanceof SyntaxError ? 400 : 500, // Bad JSON or other server error
    });
  }
});

// Example shared util (conceptual)
// async function sendConfirmationEmail(preAlertData: any): Promise<void> {
//   console.log(`Simulating email confirmation for pre-alert: ${preAlertData.id}`);
//   // Actual email sending logic here (e.g. using Supabase Realtime or a third-party email service)
//   await new Promise(resolve => setTimeout(resolve, 1000)); // Simulate async work
//   console.log(`Email supposedly sent for ${preAlertData.id}`);
// }

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lpenco510) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
