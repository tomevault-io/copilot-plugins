## cursor-rules

> Guidelines for creating and maintaining Cursor rules to ensure consistency and effectiveness.

- **Required Rule Structure:**
  ```markdown
  ---
  description: Clear, one-line description of what the rule enforces
  globs: path/to/files/*.ext, other/path/**/*
  alwaysApply: boolean
  ---

  - **Main Points in Bold**
    - Sub-points with details
    - Examples and explanations
  ```

- **File References:**
  - Use `[filename](mdc:path/to/file)` ([filename](mdc:filename)) to reference files
  - Example: [prisma.mdc](mdc:.cursor/rules/prisma.mdc) for rule references
  - Example: [schema.prisma](mdc:prisma/schema.prisma) for code references

- **Code Examples:**
  - Use language-specific code blocks
  ```typescript
  // ✅ DO: Show good examples
  const goodExample = true;

  // ❌ DON'T: Show anti-patterns
  const badExample = false;
  ```

- **Rule Content Guidelines:**
  - Start with high-level overview
  - Include specific, actionable requirements
  - Show examples of correct implementation
  - Reference existing code when possible
  - Keep rules DRY by referencing other rules

- **Rule Maintenance:**
  - Update rules when new patterns emerge
  - Add examples from actual codebase
  - Remove outdated patterns
  - Cross-reference related rules

- **Best Practices:**
  - Use bullet points for clarity
  - Keep descriptions concise
  - Include both DO and DON'T examples
  - Reference actual code over theoretical examples
  - Use consistent formatting across rules

- **Environment Variable Loading (Production & Amplify/AWS Lambda Ready)**
  - Lazily load environment variables inside functions, not at module top-level
  - Use a helper function to check for multiple prefixes (e.g., `AMPLIFY_`, `AWS_AMPLIFY_`, and no prefix)
  - Support both server and client contexts (Next.js config and `process.env`)
  - Example: See `getStripeEnvVar` in [`src/lib/stripe/init.ts`](mdc:src/lib/stripe/init.ts)

- **Next.js Environment Variables in Production (AWS Amplify)**
  - **CRITICAL**: All environment variables must be declared in the `env` section of [`next.config.mjs`](mdc:next.config.mjs) to be available at runtime in production
  - Environment variables set in AWS Amplify console are not automatically available unless explicitly declared in Next.js config
  - **AWS Amplify Environment Variable Pattern**: AWS Amplify prefixes environment variables with `AMPLIFY_` in the runtime, even if you set them without the prefix in the console
  - **NEXT_PUBLIC_ Variables in AWS Amplify**: `NEXT_PUBLIC_` prefixed variables may not be available in server-side contexts (API routes, server components) in AWS Amplify production environment
  - ✅ DO: Use `AMPLIFY_` prefixed variables for AWS Amplify deployments with fallbacks for local development
  ```javascript
  // next.config.mjs
  env: {
    // API JWT credentials - prioritize AMPLIFY_ prefix for AWS Amplify
    API_JWT_USER: process.env.AMPLIFY_API_JWT_USER || process.env.API_JWT_USER,
    API_JWT_PASS: process.env.AMPLIFY_API_JWT_PASS || process.env.API_JWT_PASS,
    AMPLIFY_API_JWT_USER: process.env.AMPLIFY_API_JWT_USER,
    AMPLIFY_API_JWT_PASS: process.env.AMPLIFY_API_JWT_PASS,
    // Keep fallbacks for local development
    NEXT_PUBLIC_API_JWT_USER: process.env.NEXT_PUBLIC_API_JWT_USER,
    NEXT_PUBLIC_API_JWT_PASS: process.env.NEXT_PUBLIC_API_JWT_PASS,
  }
  ```
  ```typescript
  // Helper functions should prioritize AMPLIFY_ prefix
  export function getApiJwtUser() {
    return (
      process.env.AMPLIFY_API_JWT_USER ||
      process.env.API_JWT_USER ||
      process.env.NEXT_PUBLIC_API_JWT_USER
    );
  }
  ```
  - ❌ DON'T: Rely only on `NEXT_PUBLIC_` prefixed variables for server-side authentication in AWS Amplify
  - ❌ DON'T: Only set environment variables in AWS Amplify console without declaring them in Next.js config
  - **Rationale**: AWS Amplify's runtime environment behavior differs from standard Next.js deployments. `AMPLIFY_` prefixed variables are reliable in production while `NEXT_PUBLIC_` variables may be unavailable in server contexts
  - **Debugging**: Variables will show as `UNDEFINED` in production logs if not declared in config or using wrong prefix pattern

- **DTO (Data Transfer Object) Setup and Usage**
  - **Centralize DTO Definitions**
    - Define all DTOs in `src/types/index.ts` (or submodules if the file grows large)
    - Use TypeScript `interface` or `type` for DTOs, matching backend API schema
    ```typescript
    // ✅ DO: Centralize DTOs
    export interface UserProfileDTO {
      id: number | null;
      userId: string;
      firstName: string;
      lastName: string;
      email: string;
      // ...other fields
    }
    ```
  - **Keep DTOs Flat and Serializable**
    - Avoid methods or computed properties; use only serializable types
    ```typescript
    // ✅ DO: Use only serializable fields
    export interface SubscriptionDTO {
      id: number | null;
      userId: string;
      plan: string;
      status: 'active' | 'inactive' | 'canceled';
      // ...other fields
    }
    ```
  - **Match Backend Schema**
    - Align DTO fields/types with backend API (OpenAPI/Swagger, Prisma, REST docs)
    - Document intentional differences
    ```typescript
    // ✅ DO: Match backend schema
    export interface EventDTO {
      id: number;
      name: string;
      date: string; // ISO string
      // ...other fields
    }
    ```
  - **Use DTOs in All API Calls and Forms**
    - Import DTOs wherever you handle API data (fetch, mutate, form state, validation)
    - Type API responses and form state with DTOs
    ```typescript
    // ✅ DO: Use DTOs in API and forms
    import type { UserProfileDTO } from "@/types";
    const profile: UserProfileDTO = await fetchProfile();
    const [formData, setFormData] = useState<Omit<UserProfileDTO, 'createdAt' | 'updatedAt'>>(defaultFormData);
    ```
  - **Extend or Compose DTOs for Feature-Specific Needs**
    - Extend DTOs for feature-specific variants
    - Use TypeScript utility types (`Pick`, `Omit`, `Partial`)
    ```typescript
    // ✅ DO: Compose DTOs for feature needs
    export type UserProfileFormDTO = Omit<UserProfileDTO, 'createdAt' | 'updatedAt'>;
    export interface UserProfileWithSubscriptionDTO extends UserProfileDTO {
      subscription?: SubscriptionDTO;
    }
    ```
  - **Document DTOs**
    - Add JSDoc comments to each DTO for clarity
    ```typescript
    /**
     * DTO for user profile data exchanged with the backend.
     */
    export interface UserProfileDTO {
      // ...
    }
    ```
  - **Update DTOs When Backend Changes**
    - Review and update DTOs whenever the backend schema changes
    - Refactor usages across the project to match updated DTOs

- **API Invocation & Data Processing Guidelines**
  - Use server components or route handlers for initial data fetching (e.g., Crust APIs)
  - Always `await` data before rendering the page/component
    ```typescript
    // In a server component
    const data = await fetchCrustApi();
    return <Page data={data} />;
    ```
  - Handle headers only in server context (server components, route handlers, middleware)
  - Pass data to client components via props
  - Gracefully handle API errors (show fallback UI or error message)

- **Summary Table**

| Guideline                      | Example/Note                                 |
|-------------------------------|----------------------------------------------|
| Centralize DTOs                | src/types/index.ts                           |
| Keep DTOs flat/serializable    | No methods, only data fields                 |
| Match backend schema           | Align field names/types                      |
| Use DTOs in API/forms          | Type API responses, form state               |
| Compose/extend for features    | Omit, Pick, extends                          |
| Document DTOs                  | JSDoc comments                               |
| Update on backend changes      | Refactor usages when backend changes         |
| Lazy env var loading           | Use helper, see src/lib/stripe/init.ts       |
| API fetch in server context    | Use server components/route handlers         |
| Await data before rendering    | Always await before rendering                |
| Handle headers in server only  | Never in client components                   |
| Pass data via props            | Server → client via props                    |
| Graceful API error handling    | Fallback UI, error message                   |

- **Rule Maintenance:**
  - Update rules when new patterns emerge
  - Add examples from actual codebase
  - Remove outdated patterns
  - Cross-reference related rules

- **Best Practices:**
  - Use bullet points for clarity
  - Keep descriptions concise
  - Include both DO and DON'T examples
  - Reference actual code over theoretical examples
  - Use consistent formatting across rules

- **All backend API calls from the frontend must use authenticated proxy endpoints**
  - Never call backend API URLs (e.g., `/api/user-profiles/...`) directly from the frontend
  - Always use the corresponding proxy route (e.g., `/api/proxy/user-profiles/...`)
  - The proxy handler is responsible for obtaining and caching the JWT token (see [`src/lib/api/jwt.ts`](mdc:src/lib/api/jwt.ts))
  - This ensures:
    - All requests are authenticated with a valid JWT
    - Token caching is handled centrally (no repeated authentication calls)
    - Security and control flow are consistent across the app

- **Rationale**
  - Prevents leaking secrets or tokens to the client
  - Ensures all API requests are properly authenticated
  - Reduces backend load by caching tokens
  - Centralizes error handling and logging

- **Correct Implementation Example**
  ```typescript
  // ✅ DO: Use the proxy endpoint for user profile fetch
  const response = await fetch(`/api/proxy/user-profiles/by-user/${userId}`, { ... });
  // ✅ DO: Use the proxy endpoint for user subscriptions
  const response = await fetch(`/api/proxy/user-subscriptions/by-profile/${profileId}`, { ... });
  ```

- **Anti-patterns**
  ```typescript
  // ❌ DON'T: Call backend API directly from the frontend
  const response = await fetch(`${apiBaseUrl}/api/user-profiles/by-user/${userId}`, { ... });
  ```

- **References**
  - See [`src/components/ProfileForm.tsx`](mdc:src/components/ProfileForm.tsx) for correct usage
  - See [`src/app/pricing/page.tsx`](mdc:src/app/pricing/page.tsx) for correct usage
  - Proxy handler: [`src/pages/api/proxy/user-profiles/[...slug].ts`](mdc:src/pages/api/proxy/user-profiles/[...slug].ts)
  - JWT caching: [`src/lib/api/jwt.ts`](mdc:src/lib/api/jwt.ts)

- **Always use absolute URLs for proxy API fetches in server components/pages**
  - In server components/pages (e.g., `src/app/pricing/page.tsx`), use an absolute URL for all fetches to `/api/proxy/...` endpoints.
  - Use `process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000'` as the base URL.
  - In client components, relative URLs (e.g., `/api/proxy/user-profiles/by-user/${userId}`) are fine.

- **Rationale**
  - Server-side fetch requires an absolute URL; relative URLs will throw a "Failed to parse URL" error.
  - Ensures consistent, working API calls in all environments.

- **Correct Implementation Example (Server Component)**
  ```typescript
  // ✅ DO: Use absolute URL in server component
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';
  const response = await fetch(`${baseUrl}/api/proxy/user-profiles/by-user/${userId}`, { ... });
  ```

- **Correct Implementation Example (Client Component)**
  ```typescript
  // ✅ DO: Use relative URL in client component
  const response = await fetch(`/api/proxy/user-profiles/by-user/${userId}`, { ... });
  ```

- **Anti-pattern**
  ```typescript
  // ❌ DON'T: Use relative URL in server component
  const response = await fetch(`/api/proxy/user-profiles/by-user/${userId}`, { ... }); // Will fail on server
  ```

- **References**
  - See [`src/app/pricing/page.tsx`](mdc:src/app/pricing/page.tsx) for correct server usage
  - See [`src/components/ProfileForm.tsx`](mdc:src/components/ProfileForm.tsx) for correct client usage

- **If a page/server component requires user authentication or session, do NOT include its route in publicPaths in middleware**
  - Only include routes in `publicPaths` if they are truly public and do not need user session data.
  - For pages like `/pricing` that need user profile or subscription info, remove them from `publicPaths` so Clerk's middleware enforces authentication.
  - This ensures the session is available in server components/pages and prevents temporary sign-outs or missing session issues.

- **Rationale**
  - Clerk's middleware only attaches the session to requests for protected routes.
  - If a route is public, server components/pages will not have access to the user session, even if the user is logged in on the client.
  - Ensures consistent authentication and session availability for all user-specific pages.

- **Correct Implementation Example**
  ```typescript
  // ✅ DO: Remove '/pricing(.*)' from publicPaths if pricing page needs user session
  const publicPaths = [
    '/',
    '/sign-in(.*)',
    '/sign-up(.*)',
    '/event(.*)',
    // '/pricing(.*)',   // REMOVED to require auth
    '/api/webhooks(.*)',
    '/api/stripe/event-checkout',
  ];
  ```

- **Anti-pattern**
  ```typescript
  // ❌ DON'T: Include user-specific pages in publicPaths
  const publicPaths = [
    '/',
    '/sign-in(.*)',
    '/sign-up(.*)',
    '/event(.*)',
    '/pricing(.*)',   // Will cause session to be unavailable in server components
    '/api/webhooks(.*)',
    '/api/stripe/event-checkout',
  ];
  ```

- **References**
  - See [`src/middleware.ts`](mdc:src/middleware.ts) for correct usage
  - See [`src/app/pricing/page.tsx`](mdc:src/app/pricing/page.tsx) for a page that requires authentication

- **For every backend API resource accessed via proxy, a corresponding proxy handler (slug route) must exist in src/pages/api/proxy/**
  - If you call `/api/proxy/resource/...` from the frontend, ensure `src/pages/api/proxy/resource/[...slug].ts` exists.
  - The proxy handler is responsible for authentication, JWT caching, and forwarding requests to the backend.
  - This ensures all API calls are authenticated, routed, and debuggable.

- **Rationale**
  - Prevents silent 404s and debugging confusion.
  - Ensures all API calls are consistently authenticated and proxied.
  - Centralizes error handling and logging for backend API calls.

- **Correct Implementation Example**
  ```typescript
  // ✅ DO: If you call /api/proxy/user-subscriptions/by-profile/:id, ensure this exists:
  // src/pages/api/proxy/user-subscriptions/[...slug].ts
  export default async function handler(req, res) { /* ... */ }
  ```

- **Anti-pattern**
  ```typescript
  // ❌ DON'T: Call /api/proxy/resource/... without a corresponding proxy handler
  // This will result in a 404 and no backend call
  ```

- **References**
  - See [`src/pages/api/proxy/user-profiles/[...slug].ts`](mdc:src/pages/api/proxy/user-profiles/[...slug].ts) for a working example
  - See [`src/pages/api/proxy/user-subscriptions/[...slug].ts`](mdc:src/pages/api/proxy/user-subscriptions/[...slug].ts) for a matching implementation

- **Never make API calls with undefined or null parameters**
  - Always guard effects and handlers to ensure required values (like userId) are present before making API calls.
  - Prevents 404s, backend errors, and unnecessary requests.
  - Example:
    ```typescript
    if (!userId) return; // Prevents bad API calls
    ```
  - See `ProfileBootstrapper.tsx` for correct usage.
  - Review all API call sites for proper guards.

- **Rationale**
  - Avoids backend errors and noisy logs
  - Prevents accidental creation of bad data or error states
  - Ensures only valid, intentional requests are made

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
