## sky-hr-web

> Use the latest version of Shadcn to install new components, like this command to add a button component:

# shadcn instructions

Use the latest version of Shadcn to install new components, like this command to add a button component:

```bash
pnpx shadcn@latest add button
```

# TanStack Start Server Functions and Middlewares

## Server Functions Pattern

When creating server functions in `src/server/*.server.ts`, follow this EXACT pattern:

```typescript
export const myServerFunction = createServerFn({
  method: "GET",  // Always use GET for server functions
})
.handler(async () => {
  const request = getRequest();
  if (!request) {
    throw new Error("Request context not available.");
  }
  const { headers } = request;

  // All authClient calls MUST use fetchOptions: { headers } to forward cookies
  const result = await authClient.someMethod({
    // ... parameters
    fetchOptions: { headers },
  });

  return result;
});
```

### Rules:
1. **Always use `method: "GET"`** - Never use POST or other methods
2. **Handler takes NO parameters** - Use `async () => { ... }`, not `async (param) => { ... }`
3. **Always get request context** - Use `getRequest()` to access headers
4. **Always forward headers** - Pass `fetchOptions: { headers }` to ALL `authClient` calls
5. **No direct imports in route files** - Import server functions, not `authClient` or `getRequest` directly in routes

## Middleware Pattern

When creating middlewares in `src/middleware/*.middleware.ts`, follow this EXACT pattern:

```typescript
export const myMiddleware = createMiddleware({
  type: "function",
}).server(async ({ next }) => {
  const request = getRequest();
  if (!request) {
    throw new Error("Request context not available.");
  }
  const { headers } = request;

  // Perform operations with headers
  const data = await authClient.someMethod({
    fetchOptions: { headers },
  });

  // Pass data to context
  return next({ context: { data } });
});
```

### Rules:
1. **Use `type: "function"`** - Always specify this
2. **Use `.server()` method** - Not `.client()` or other variants
3. **Always get request context** - Use `getRequest()` to access headers
4. **Always forward headers** - Pass `fetchOptions: { headers }` to ALL `authClient` calls
5. **Return `next({ context: { ... } })`** - Pass data via context

## Using Server Functions in Routes

In route `beforeLoad` or components:

```typescript
// ✅ CORRECT - Import and call server function
import { myServerFunction } from '@/server/my.server';

export const Route = createFileRoute('/my-route')({
  beforeLoad: async () => {
    const result = await myServerFunction();
    // Use result...
  },
});

// ❌ WRONG - Don't use getRequest() or authClient directly in routes
import { getRequest } from '@tanstack/react-start/server';
import { authClient } from '@/lib/auth-client';
```

## Why This Pattern?

1. **Cookie forwarding** - Headers contain cookies that must be forwarded from SSR server to API
2. **Railway/production compatibility** - Works with separate subdomains and HTTPS
3. **Consistency** - All existing server functions follow this exact pattern
4. **Type safety** - TanStack Start handles serialization automatically

## Examples from Codebase

- ✅ `getUserOrganizations()` - Gets organizations with headers
- ✅ `getUserInvitations()` - Gets invitations with headers  
- ✅ `setFirstOrganizationActive()` - Sets active org with headers
- ✅ `authMiddleware` - Gets session with headers
- ✅ `organizationMiddleware` - Gets organization with headers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yamilyscapa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
