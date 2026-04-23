## mailgun-edge-examples

> Rules for Netlify Edge Functions examples under netlify/


# Platform: Netlify Edge Functions

Runtime: **Deno** (same as Supabase!)

## File Structure

Follow this pattern for all examples:

```
netlify/
├── edge-functions/
│   └── example-name.ts      # Edge function code
├── netlify.toml             # Configuration
└── README.md
```

## Edge Function Format

Export default handler with config:

```typescript
import type { Context } from "https://edge.netlify.com"

export default async (request: Request, context: Context) => {
  try {
    // 1. Parse request
    const { param } = await request.json()

    // 2. Get API credentials
    const apiKey = Deno.env.get('API_KEY')
    if (!apiKey) {
      throw new Error('API_KEY not configured')
    }

    // 3. Call external API
    const response = await fetch('https://api.example.com/endpoint', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey}`,
      },
      body: JSON.stringify({ param }),
    })

    if (!response.ok) {
      throw new Error(`API error: ${response.statusText}`)
    }

    const data = await response.json()

    // 4. Return result
    return new Response(
      JSON.stringify(data),
      {
        status: 200,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        },
      },
    )
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      {
        status: 400,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        },
      },
    )
  }
}

export const config = { 
  path: "/api/example-name",
  method: ["GET", "POST"]
}
```

## Configuration: netlify.toml

Create or update `netlify.toml`:

```toml
[build]
  publish = "public"

[[edge_functions]]
  function = "example-name"
  path = "/api/example-name"
```

## Environment Variables

Access via `Deno.env.get()` (same as Supabase):

```typescript
const apiKey = Deno.env.get('MAILGUN_API_KEY')
const domain = Deno.env.get('MAILGUN_DOMAIN')
```

Set via Netlify Dashboard or CLI:
```bash
netlify env:set MAILGUN_API_KEY key-xxxxx
```

For local development, create `.env`:
```bash
MAILGUN_API_KEY=key-xxxxx
MAILGUN_DOMAIN=mg.example.com
```

## CORS Handling

Add CORS headers to all responses:

```typescript
headers: {
  'Content-Type': 'application/json',
  'Access-Control-Allow-Origin': '*',
}
```

For preflight requests, handle in the function:

```typescript
if (request.method === 'OPTIONS') {
  return new Response(null, {
    status: 204,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
      'Access-Control-Allow-Headers': 'content-type, authorization',
    },
  })
}
```

## Context Object

Netlify provides helpful context:

```typescript
export default async (request: Request, context: Context) => {
  // Geolocation
  const { city, country } = context.geo

  // Cookies
  const sessionId = context.cookies.get('session')

  // Logging (appears in function logs)
  context.log('Processing request from', country)

  // Access site configuration
  const siteUrl = context.site.url

  // Continue to next handler (middleware pattern)
  return context.next()
}
```

## README Template

```markdown
# Function Name

Brief description of what this does.

## External API

- **Service**: [API Name](https://docs-url.com)
- **Endpoint**: `POST /v1/endpoint`
- **What it does**: One sentence

## Setup

1. Get API key from [Service Dashboard](https://example.com)

2. Set environment variables:
   ```bash
   netlify env:set API_KEY your_key_here
   ```

3. Deploy:
   ```bash
   netlify deploy --prod
   ```

## Usage

```bash
curl -X POST 'https://your-site.netlify.app/api/function-name' \
  -H 'Content-Type: application/json' \
  -d '{
    "param": "value"
  }'
```

## Response

```json
{
  "result": "success"
}
```

## Local Development

```bash
netlify dev
# Function available at http://localhost:8888/api/function-name
```
```

## Testing Locally

```bash
# Start Netlify dev server (loads .env automatically)
netlify dev

# Test in another terminal
curl -X POST 'http://localhost:8888/api/function-name' \
  -H 'Content-Type: application/json' \
  -d '{"test": "data"}'
```

## Common Patterns

### FormData for APIs that require it:

```typescript
const formData = new FormData()
formData.append('from', 'sender@example.com')
formData.append('to', 'recipient@example.com')
formData.append('subject', 'Hello')
formData.append('text', 'Message body')

const response = await fetch(
  `https://api.mailgun.net/v3/${Deno.env.get('MAILGUN_DOMAIN')}/messages`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${btoa(`api:${Deno.env.get('MAILGUN_API_KEY')}`)}`,
    },
    body: formData,
  }
)
```

### Handle different HTTP methods:

```typescript
export default async (request: Request, context: Context) => {
  if (request.method === 'GET') {
    const url = new URL(request.url)
    const email = url.searchParams.get('email')
    // Handle GET
  }

  if (request.method === 'POST') {
    const body = await request.json()
    // Handle POST
  }
}

export const config = {
  path: "/api/function-name",
  method: ["GET", "POST"]
}
```

### Middleware pattern (transform responses):

```typescript
export default async (request: Request, context: Context) => {
  // Let the request continue to other handlers
  const response = await context.next()

  // Transform the response
  const body = await response.text()
  const modified = body.replace('foo', 'bar')

  return new Response(modified, {
    headers: response.headers,
    status: response.status,
  })
}
```

### Geolocation-based responses:

```typescript
export default async (request: Request, context: Context) => {
  const { country, city } = context.geo

  return new Response(
    JSON.stringify({ 
      message: `Hello from ${city}, ${country}!` 
    }),
    { headers: { 'Content-Type': 'application/json' } }
  )
}
```

## Importing Dependencies

Use Deno-style imports:

```typescript
// npm packages via esm.sh
import { parse } from "https://esm.sh/marked@4"

// Netlify edge context types
import type { Context } from "https://edge.netlify.com"
```

## What NOT to Include

- Heavy dependencies (keep bundles small)
- Node.js-specific APIs (use Deno/Web APIs)
- Long-running operations
- Database queries without connection pooling

## When Creating New Examples

1. Check `specs/` for requirements
2. Create file: `netlify/edge-functions/example-name.ts`
3. Copy template above
4. Update `netlify.toml` with function config
5. Implement the specific API integration
6. Create README with complete documentation
7. Test locally with `netlify dev`
8. Keep it simple - one API call, clear pattern

## Deployment

```bash
# Deploy preview
netlify deploy

# Deploy to production
netlify deploy --prod
```

## Key Differences from Supabase

- ✓ Same Deno runtime
- ✓ Same `Deno.env.get()` for secrets
- ✓ Same import patterns (esm.sh, npm:)
- ✗ Different config (netlify.toml vs CLI)
- ✗ Context object with geo, cookies, etc.
- ✗ Middleware pattern with `context.next()`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/djbriane) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
