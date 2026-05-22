## nextjs-api-routes

> Rules for Next.js API route structure and security


- **All API routes must...**
  - Use authentication middleware
  - Return JSON responses
  - etc.
- **No local DTO/interface declarations**
  - All DTOs should be imported from `@/types` if needed (none should be declared locally)
  - No DTO redeclaration: All DTOs should be imported from `@/types` (none needed in this handler)
  - Example:
    ```typescript
    // ✅ DO: Import DTOs
    import type { UserProfileDTO } from '@/types';
    // ❌ DON'T: Redeclare DTOs
    // interface UserProfileDTO { ... }
    ```

- **Consistent environment variable**
  - Use `process.env.NEXT_PUBLIC_API_BASE_URL` for the backend API base URL, matching the rest of your proxy routes
  - Do not use hardcoded URLs or other env vars for backend base URL
  - Example:
    ```typescript
    const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
    ```

- **Query string handling**
  - Use a `buildQueryString` helper to forward all query params, just like in the user profile proxy
  - Ensures all filters, pagination, and sorts are preserved
  - Example:
    ```typescript
    function buildQueryString(query: Record<string, any>) {
      const params = new URLSearchParams();
      for (const key in query) {
        const value = query[key];
        if (Array.isArray(value)) {
          value.forEach(v => params.append(key, v));
        } else if (typeof value !== 'undefined') {
          params.append(key, value);
        }
      }
      return params.toString();
    }
    ```

- **Do NOT add tenantId.equals in your client/server code when calling the proxy**
  - The proxy handler will always inject tenantId.equals automatically.
  - Only add tenantId.equals if you are calling the backend API directly (not via /api/proxy/...).
  - This prevents duplicate tenantId.equals parameters and backend criteria errors.
  - Example:
    ```typescript
    // ✅ DO: Only add email.equals or userId.equals
    const params = new URLSearchParams({ 'email.equals': email });
    await fetch('/api/proxy/user-profiles?' + params.toString());
    // ❌ DON'T: Add tenantId.equals when calling the proxy
    const params = new URLSearchParams({ 'email.equals': email, 'tenantId.equals': tenantId });
    await fetch('/api/proxy/user-profiles?' + params.toString()); // Will result in duplicate tenantId
    ```

- **JWT handling**
  - Use `fetchWithJwtRetry` **from `@/lib/proxyHandler`** for all backend calls (it adds **`Authorization`** and **`X-Tenant-ID`**; see **X-Tenant-ID for tenant-scoped backend APIs** below). Older inline examples may omit `X-Tenant-ID` — do not copy them for direct backend URLs.
  - Do not call backend APIs directly with fetch; always use the helper
  - Example:
    ```typescript
    import { getCachedApiJwt, generateApiJwt } from '@/lib/api/jwt';
    async function fetchWithJwtRetry(apiUrl: string, options: any = {}, debugLabel = '') {
      let token = await getCachedApiJwt();
      let response = await fetch(apiUrl, {
        ...options,
        headers: {
          ...options.headers,
          Authorization: `Bearer ${token}`,
        },
      });
      if (response.status === 401) {
        token = await generateApiJwt();
        response = await fetch(apiUrl, {
          ...options,
          headers: {
            ...options.headers,
            Authorization: `Bearer ${token}`,
          },
        });
      }
      return response;
    }
    ```

- **Error handling**
  - Catch and log errors, returning a 500 with a clear message if something goes wrong
  - Example:
    ```typescript
    try {
      // ...
    } catch (err) {
      console.error('Proxy error:', err);
      res.status(500).json({ error: 'Internal server error', details: String(err) });
    }
    ```

- **Method handling**
  - Only allow GET and POST (or appropriate methods), with proper 405 responses for others
  - Example:
    ```typescript
    if (req.method === 'GET') { /* ... */ }
    else if (req.method === 'POST') { /* ... */ }
    else {
      res.setHeader('Allow', ['GET', 'POST']);
      res.status(405).end(`Method ${req.method} Not Allowed`);
    }
    ```

- **Required backend fields for create operations**
  - When creating resources via proxy API routes, all fields required by the backend (including timestamps like `createdAt` and `updatedAt`) must be included in the payload, even if not set by the client.
  - The proxy or client must ensure these fields are present to avoid backend validation errors (e.g., Spring Boot will reject null `createdAt`/`updatedAt`).
  - Example for ticket type creation:
    ```typescript
    // ✅ DO: Include all required fields
    const payload = {
      ...form,
      eventId,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };
    await fetch('/api/proxy/ticket-types', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    ```

- **ID field handling for create operations**
  - For POST (create) operations, do not include the 'id' field in the payload (or set it to null if required by the backend).
  - Only include 'id' for update (PUT/PATCH) operations.
  - This matches backend expectations and avoids sending unnecessary or misleading ids during creation.
  - Example for ticket type creation:
    ```typescript
    // ✅ DO: Omit 'id' for create
    const { id, ...rest } = form;
    const payload = {
      ...rest,
      event: { id: eventId },
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };
    await fetch('/api/proxy/ticket-types', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    ```
  - **Clarification:** If the backend requires the 'id' field to be present (but null) for POST requests, explicitly set `id: null` in the payload. This is sometimes required by strict backend validation or OpenAPI schemas.
  - Example for explicit null id:
    ```typescript
    // ✅ DO: Set id: null if backend requires it
    const { id, ...rest } = form;
    const payload = {
      ...rest,
      id: null, // Explicitly set to null for backend compatibility
      event: { id: eventId },
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };
    await fetch('/api/proxy/ticket-types', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    ```

- **Pattern Used**
  - This matches the pattern in:
    - `src/pages/api/proxy/user-profiles/index.ts`
    - `src/components/ProfileForm.tsx`
    - This rule file itself

- **Correct Implementation Example**
  ```typescript
  // src/pages/api/proxy/ticket-types/index.ts
  import type { NextApiRequest, NextApiResponse } from 'next';
  import { getCachedApiJwt, generateApiJwt } from '@/lib/api/jwt';
  const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
  function buildQueryString(query: Record<string, any>) { /* ... */ }
  async function fetchWithJwtRetry(apiUrl: string, options: any = {}, debugLabel = '') { /* ... */ }
  export default async function handler(req: NextApiRequest, res: NextApiResponse) {
    try {
      if (req.method === 'GET') {
        const qs = buildQueryString(req.query);
        const apiUrl = `${API_BASE_URL}/api/ticket-types${qs ? `?${qs}` : ''}`;
        const response = await fetchWithJwtRetry(apiUrl, { method: 'GET' });
        const data = await response.json();
        res.status(response.status).json(data);
      } else if (req.method === 'POST') {
        const apiUrl = `${API_BASE_URL}/api/ticket-types`;
        const response = await fetchWithJwtRetry(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(req.body),
        });
        const data = await response.json();
        res.status(response.status).json(data);
      } else {
        res.setHeader('Allow', ['GET', 'POST']);
        res.status(405).end(`Method ${req.method} Not Allowed`);
      }
    } catch (err) {
      console.error('Proxy error:', err);
      res.status(500).json({ error: 'Internal server error', details: String(err) });
    }
  }
  ```

- **Anti-patterns**
  ```typescript
  // ❌ DON'T: Redeclare DTOs or use hardcoded URLs
  interface TicketTypeDTO { ... }
  const API_BASE_URL = 'http://localhost:8080';
  // ❌ DON'T: Call fetch directly without JWT helper
  const response = await fetch(apiUrl, { ... });
  ```

- **Rule Maintenance**
  - Update this rule when new patterns emerge
  - Add examples from actual codebase
  - Remove outdated patterns
  - Cross-reference related rules

- **Automated tenantId injection in DTOs**
  - Always include the `tenantId` field in every DTO sent to the backend. The value must come from the environment variable `NEXT_PUBLIC_TENANT_ID`.
  - Use a centralized helper to read the tenant ID (see `getTenantId` in `src/lib/env.ts`).
  - Use the `withTenantId` utility (`src/lib/withTenantId.ts`) to inject the tenantId into any DTO before sending it to the backend. This ensures consistency and prevents missing tenant IDs.
  - Never hardcode the tenantId or set it manually in multiple places.
  - Example usage:
    ```typescript
    import { withTenantId } from '@/lib/withTenantId';
    const payload = withTenantId({
      ...formData,
      // other fields
    });
    await fetch('/api/proxy/some-endpoint', {
      method: 'POST',
      body: JSON.stringify(payload),
      headers: { 'Content-Type': 'application/json' },
    });
    ```
  - The environment variable must be set in `.env.local` as:
    ```env
    NEXT_PUBLIC_TENANT_ID=your-tenant-id
    ```
  - The helper must throw a clear error if the variable is missing, to prevent silent failures.
  - This pattern is required for all multi-tenant API calls and DTOs.
  - See also: `src/lib/env.ts`, `src/lib/withTenantId.ts` for implementation details.

- **TenantId injection in all proxy API routes (cross-cutting enforcement)**
  - All proxy API routes (e.g., /api/proxy/event-details, /api/proxy/ticket-types, etc.) must use the shared `createProxyHandler` from `src/lib/proxyHandler.ts`.
  - This handler automatically injects `tenantId` into all DTOs for POST/PUT/PATCH requests using the `withTenantId` utility.
  - Example usage:
    ```typescript
    import { createProxyHandler } from '@/lib/proxyHandler';
    export default createProxyHandler({ backendPath: '/api/event-details' });
    ```
  - **Rationale:**
    - Guarantees that every create/update request includes the correct tenantId, regardless of frontend implementation.
    - Prevents accidental omission of tenantId in multi-tenant environments.
    - Centralizes error handling, JWT logic, and query string forwarding.
  - **Backend enforcement:**
    - The backend (Rust API) should also validate that tenantId is present in every DTO and reject requests if missing.
  - See also: `withTenantId` utility, Rust validation snippet below.

- **Add `[...slug].ts` proxies for single resource operations using shared handler**
  - For every backend resource that supports single-resource operations (e.g., GET/PUT/DELETE by ID), add a `[...slug].ts` file in the corresponding proxy API directory.
  - The handler **must** use the shared `createProxyHandler` from `@/lib/proxyHandler`, passing the correct `backendPath` (e.g., `/api/event-details`).
  - **Remove all custom logic and unused imports** from these handlers; the shared handler centralizes JWT, tenantId, error, and query param logic.
  - **Examples:**
    ```typescript
    // src/pages/api/proxy/event-details/[...slug].ts
    import { createProxyHandler } from '@/lib/proxyHandler';
    export default createProxyHandler({ backendPath: '/api/event-details' });

    // src/pages/api/proxy/event-medias/[...slug].ts
    import { createProxyHandler } from '@/lib/proxyHandler';
    export default createProxyHandler({ backendPath: '/api/event-medias' });
    ```
  - This ensures all single-resource proxy routes are DRY, secure, and multi-tenant aware by default.
  - **Rationale:**
    - Prevents code duplication and errors in per-route logic
    - Guarantees tenantId injection and JWT handling for all resource operations
    - Simplifies maintenance and onboarding for new resources

- **JHipster/Spring Data REST filter syntax for criteria queries**
  - When calling backend APIs that use JHipster or Spring Data REST criteria objects, always use the correct filter syntax: `field.operation=value` (e.g., `tenantId.equals=tenant_demo_001`).
  - Do **not** use just `tenantId=...` or `field=...` for filter fields; this will cause type conversion errors in the backend.
  - Common operations: `.equals`, `.contains`, `.in`, etc. (e.g., `userStatus.equals=ACTIVE`, `email.contains=gmail.com`)
  - Example:
    ```typescript
    // ✅ DO: Use .equals for exact match
    params.append('tenantId.equals', getTenantId());
    // ✅ DO: Use .contains for substring match
    params.append('email.contains', 'gmail.com');
    // ❌ DON'T: Use just tenantId=...
    params.append('tenantId', getTenantId()); // Will cause backend error
    ```
  - This applies to all criteria-based GET endpoints, especially for multi-tenant filtering and user/resource queries.
  - See also: UserProfileCriteria in backend code for supported filters.

- **All authenticated fetches must be server-side**
  - Never fetch authenticated resources (e.g., user profiles, protected APIs) directly from the client. Always perform these fetches in a server component, server action, or API route.
  - Rationale: Only server-side code has access to the user's session and can generate/attach a valid JWT. Client-side fetches will not have the session and will result in 401 Unauthorized errors.
  - Example:
    ```typescript
    // ✅ DO: Fetch admin profile server-side
    export default async function ManageUsagePage() {
      const { userId } = auth();
      const adminProfile = userId ? await fetchAdminProfileServer(userId) : null;
      // ...
    }
    // ❌ DON'T: Fetch admin profile in useEffect or client-side hooks
    useEffect(() => {
      fetch('/api/proxy/user-profiles/by-user/' + userId);
    }, [userId]);
    ```
  - See also: ProfileBootstrapper, ProfileForm, ManageUsagePage for correct patterns.

- **All authenticated API calls must be made from server actions or server components, not from client components**
  - Never call protected proxy API endpoints (e.g., /api/proxy/user-profiles, /api/proxy/ticket-types, etc.) directly from client components.
  - Always create a server action (e.g., actions.ts) or use a server component to perform the API call, then pass the result to the client component as props or via server action invocation.
  - This ensures JWT/session is available, tenantId is injected, and security is enforced.
  - Example:
    ```typescript
    // src/app/admin/manage-usage/actions.ts
    export async function patchUserProfileServer(userId: number, payload: any) { /* ... */ }
    // src/app/admin/manage-usage/ManageUsageClient.tsx
    import { patchUserProfileServer } from './actions';
    // ...
    await patchUserProfileServer(user.id, payload);
    ```
  - See also: ProfileBootstrapper, ProfileForm, ManageUsagePage for correct patterns.

- **Client Components Must Not Make Direct API Calls**
  - Client components (marked with 'use client') must NEVER make direct fetch calls to API endpoints.
  - This includes both proxy endpoints (/api/proxy/...) and direct backend calls.
  - Client components should only:
    - Receive data as props from server components
    - Call server actions for mutations
    - Handle UI state and user interactions
  - **Rationale:** Client components run in the browser where:
    - Environment variables may not be available
    - JWT tokens and session data are not accessible
    - Direct API calls will fail with authentication errors
  - **Correct Pattern:**
    ```typescript
    // ✅ DO: Server component fetches data
    // src/app/profile/page.tsx (server component)
    export default async function ProfilePage() {
      const profile = await fetchProfileServer(userId);
      return <ProfileForm profile={profile} />;
    }

    // ✅ DO: Client component receives props
    // src/components/ProfileForm.tsx (client component)
    'use client';
    export function ProfileForm({ profile }: { profile: UserProfileDTO }) {
      // Handle form state and UI interactions only
    }

    // ✅ DO: Client component calls server action
    // src/components/ProfileForm.tsx
    import { updateProfileServer } from './actions';
    const handleSubmit = async (data: any) => {
      await updateProfileServer(data);
    };
    ```
  - **Anti-patterns:**
    ```typescript
    // ❌ DON'T: Client component making direct API calls
    'use client';
    export function ProfileForm() {
      useEffect(() => {
        fetch('/api/proxy/user-profiles/by-user/' + userId);
      }, [userId]);
    }
    ```
  - **References:**
    - See `src/components/ProfileForm.tsx` for problematic implementation
    - See `src/components/DashboardContent.tsx` for problematic implementation
    - See `src/app/admin/manage-usage/ManageUsageClient.tsx` for correct pattern

- **Standard: Place all server-side API calls in ApiServerActions.ts**
  - If your module makes authenticated or protected API calls, create a file named `ApiServerActions.ts` in that folder.
  - Place all server-side API calls (fetch, patch, post, etc.) in this file as exported async functions.
  - Import and use these actions from your client components.
  - This ensures all API calls are server-side, JWT/session is available, and security is enforced.
  - Example:
    ```typescript
    // src/app/admin/manage-usage/ApiServerActions.ts
    export async function patchUserProfileServer(userId: number, payload: any) { /* ... */ }
    // src/app/admin/manage-usage/ManageUsageClient.tsx
    import { patchUserProfileServer } from './ApiServerActions';
    // ...
    await patchUserProfileServer(user.id, payload);
    ```
  - This is now the standard for all modules with API calls.

## PATCH/PUT Server Actions: Direct Backend Update Pattern (Service JWT, No Proxy)

- **For PATCH/PUT operations that update backend resources, prefer direct backend calls from server actions using a service JWT, not via the proxy, when sessionless service access is required.**
  - Use `getCachedApiJwt()` (and `generateApiJwt()` as fallback) to obtain a service JWT.
  - Always include the `id` field in the payload for PATCH/PUT, as required by backend conventions.
  - Set `Content-Type: application/merge-patch+json` for PATCH (or `application/json` for PUT if required by backend).
  - Attach the JWT as an `Authorization` header: `Bearer <token>`.
  - Do not rely on Clerk session or cookies for these calls.
  - Example:
    ```typescript
    import { getCachedApiJwt, generateApiJwt } from '@/lib/api/jwt';
    export async function patchResourceServer(resourceId: number, payload: Partial<ResourceDTO>) {
      const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
      const url = `${API_BASE_URL}/api/resource/${resourceId}`;
      let token = await getCachedApiJwt();
      if (!token) token = await generateApiJwt();
      const finalPayload = { ...payload, id: resourceId };
      const res = await fetch(url, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/merge-patch+json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(finalPayload),
      });
      if (!res.ok) throw new Error(await res.text());
      return res.json();
    }
    ```
  - See `patchUserProfileServer` and `updateEventTicketTransactionCheckIn` for real examples.
  - This pattern is required for all PATCH/PUT server actions that do not require user session context.

---
**Webhooks REST API Call Pattern (Stripe, etc.)**
- When making REST API calls from webhook files (such as Stripe webhooks in src/app/api/webhooks/stripe/route.ts), do NOT use the standard /api/proxy/... pattern or proxy API handler.
- Instead, call the backend Rust API directly from the webhook file using fetch, and:
  - Always include the JWT token in the Authorization header: 'Authorization: Bearer <token>'
  - Always pass the 'id' field and all required fields in the PATCH/PUT/POST payload, matching the backend DTO requirements.
  - Use 'Content-Type: application/merge-patch+json' for PATCH requests (or 'application/json' for POST/PUT as required).
  - Do not rely on Clerk session or cookies; use service JWT only.
  - See handleChargeFeeUpdate in src/app/api/webhooks/stripe/route.ts for a reference implementation.
- Rationale: Webhook files run in a different context and must not use the proxy API pattern. This ensures correct authentication and data integrity for backend updates.
- Project-wide convention: REST API calls from server action scripts use nextjs_api_routes.mdc. Webhook files must follow this direct-call pattern for backend updates.
---

- **Next.js 15+ Dynamic Route Async Context Rule**
  - In app router page, layout, or route handlers, always `await` any async context objects such as `params`, `headers()`, or `cookies()` before using their properties.
  - This is required in Next.js 15+ where these objects may be promises.
  - **DO:**
    ```typescript
    export default async function Page(props: { params: { id: string } }) {
      const { params } = props;
      // If params is a promise, await it
      const resolvedParams = typeof params.then === 'function' ? await params : params;
      const id = resolvedParams.id;
      // ...
    }
    ```
  - **DON'T:**
    ```typescript
    // ❌ DON'T: Use params.id directly if params may be a promise
    const id = params.id; // May throw in Next.js 15+
    ```
  - **Rationale:**
    - Prevents runtime errors like "params should be awaited before using its properties".
    - Ensures compatibility with Next.js 15+ dynamic route context.
  - See: https://nextjs.org/docs/messages/sync-dynamic-apis

- **TenantId Query Parameter and Body Injection Refinement**
  - Only add `tenantId.equals` as a query parameter for list/filter endpoints **if the REST API schema requires it** (as specified in design/requirements). Do **not** inject to every list/filter request by default.
  - Only inject `tenantId` into the request body if the DTO defines it (i.e., if the field exists in the request body object). Do **not** add `tenantId` to the body if the DTO does not have a `tenantId` field.
  - Rationale: Prevents backend errors and ensures compliance with API contracts.
  - Example:
    ```typescript
    // ✅ DO: Add tenantId.equals only if required by the API schema
    const qs = new URLSearchParams();
    if (shouldAddTenantIdEquals) qs.append('tenantId.equals', tenantId);
    // ✅ DO: Inject tenantId into body only if field exists
    if ('tenantId' in dto) dto.tenantId = tenantId;
    ```
  - See also: proxy handler implementation for conditional logic.

---
**PATCH/PUT/DELETE Proxy Handler Body Parser Rule**
- For all API proxy handlers in `[...slug].ts` (or any dynamic route handler that supports PATCH/PUT/DELETE), you **MUST** set:
  ```typescript
  export const config = {
    api: {
      bodyParser: false,
    },
  };
  ```
- This disables the default body parser, allowing raw JSON/merge-patch+json payloads to be forwarded to the backend.
- Without this, PATCH/PUT requests may hang or fail, especially for large or non-standard payloads.
- This matches the pattern in `src/pages/api/proxy/event-ticket-transactions/[...slug].ts` and is required for all similar endpoints.
- **References:**
  - See `src/pages/api/proxy/event-ticket-transactions/[...slug].ts` for a working example.
  - See `src/pages/api/proxy/discount-codes/[...slug].ts` for the required fix.

- **Port-Agnostic App URL Configuration**
  - Use `getAppUrl()` from `@/lib/env` instead of hardcoded URLs for server-side API calls.
  - This ensures the application works on any port (3000, 3001, etc.) without hardcoding.
  - **DO:**
    ```typescript
    import { getAppUrl } from '@/lib/env';
    const baseUrl = getAppUrl();
    const response = await fetch(`${baseUrl}/api/proxy/event-details`);
    ```
  - **DON'T:**
    ```typescript
    // ❌ DON'T: Hardcode port numbers
    const baseUrl = 'http://localhost:3000';
    // ❌ DON'T: Use environment variable with hardcoded fallback
    const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';
    ```
  - **Rationale:**
    - Makes the application truly port-agnostic
    - Supports development on any port (3000, 3001, 3002, etc.)
    - Maintains compatibility with production environment variables
  - **References:**
    - See `src/app/page.tsx` for correct usage
    - See `src/lib/env.ts` for the `getAppUrl()` implementation

- **Graceful API Failure Handling**
  - Always handle API fetch failures gracefully in server components to prevent page crashes.
  - Use try-catch blocks around all API calls and provide fallback data.
  - Log errors for debugging but don't let them break the user experience.
  - **DO:**
    ```typescript
    async function fetchData() {
      try {
        const response = await fetch('/api/proxy/some-endpoint');
        if (response.ok) {
          return await response.json();
        }
      } catch (error) {
        console.error('API fetch failed:', error);
      }
      return []; // Return empty array or default data
    }
    ```
  - **DON'T:**
    ```typescript
    // ❌ DON'T: Let API failures crash the page
    const data = await fetch('/api/proxy/some-endpoint');
    return data.json(); // This will throw if fetch fails
    ```
  - **Common Causes of Fetch Failures:**
    - Backend API not running
    - Missing environment variables (API credentials, tenant ID)
    - Network connectivity issues
    - Authentication failures
  - **Debugging Steps:**
    1. Check if backend API is running
    2. Verify environment variables are set
    3. Check network connectivity
    4. Review proxy route logs
  - **References:**
    - See `src/app/page.tsx` for graceful error handling example

- **Public Page API Call Pattern: Avoid Server Actions from Client Components**
  - For public pages (like homepage) that don't require authentication, avoid calling server actions from client components to prevent Next.js 15+ `headers()` async context errors.
  - Instead, use direct `fetch()` calls to proxy endpoints that are listed in the public routes.
  - **Rationale:** Server actions called from client components can trigger `headers()` without awaiting, causing runtime errors in Next.js 15+.
  - **DO:**
    ```typescript
    // ✅ DO: Use direct fetch to proxy endpoints from client components on public pages
    'use client';
    export function TeamSection() {
      useEffect(() => {
        const loadData = async () => {
          const baseUrl = getAppUrl();
          const response = await fetch(
            `${baseUrl}/api/proxy/executive-committee-team-members?isActive.equals=true&sort=priorityOrder,asc`,
            {
              method: 'GET',
              headers: { 'Content-Type': 'application/json' },
              cache: 'no-store',
            }
          );
          if (response.ok) {
            const data = await response.json();
            setData(Array.isArray(data) ? data : []);
          }
        };
        loadData();
      }, []);
    }
    ```
  - **Executive committee priority order:** The list is sorted by `priorityOrder` ascending (`sort=priorityOrder,asc`). **Lower number = higher rank** (0 appears first, 10 later). Show this tip in the admin form (tooltip or info above the Priority Order field).
  - **DON'T:**
    ```typescript
    // ❌ DON'T: Call server actions from client components on public pages
    'use client';
    export function TeamSection() {
      useEffect(() => {
        const loadData = async () => {
          const data = await fetchExecutiveTeamMembersServer(); // Server action - causes headers() error
          setData(data);
        };
        loadData();
      }, []);
    }
    ```
  - **When to Use This Pattern:**
    - Public pages (homepage, event pages, etc.) that don't require authentication
    - Client components that need to fetch data on mount
    - When the proxy endpoint is in the public routes list
  - **When NOT to Use:**
    - Protected/admin pages that require authentication
    - Server components (use server actions normally)
    - When you need JWT authentication (use server actions with proper auth)
  - **References:**
    - See `src/components/TeamSection.tsx` for correct implementation
    - See `src/components/charity-sections/TeamSection.tsx` for correct implementation
    - See `src/middleware.ts` for public routes configuration

- **STRICT RULE: All Server Actions Must Use fetchWithJwtRetry for Backend API Calls**
  - **CRITICAL**: All server actions that make backend API calls MUST use `fetchWithJwtRetry` from `@/lib/proxyHandler`.
  - **NEVER** use direct `fetch()` calls to backend APIs in server actions.
  - **NEVER** implement custom JWT retry logic in server actions.
  - **Rationale**: This ensures consistent authentication, error handling, and prevents `headers()` async context errors in Next.js 15+.
  - **DO:**
    ```typescript
    // ✅ DO: Use fetchWithJwtRetry for all backend API calls
    import { fetchWithJwtRetry } from '@/lib/proxyHandler';

    export async function fetchDataServer() {
      const res = await fetchWithJwtRetry(`${API_BASE_URL}/api/some-endpoint`, {
        cache: 'no-store',
      });
      if (!res.ok) return null;
      return await res.json();
    }
    ```
  - **DON'T:**
    ```typescript
    // ❌ DON'T: Use direct fetch with custom JWT logic
    let token = await getCachedApiJwt();
    let res = await fetch(`${API_BASE_URL}/api/some-endpoint`, {
      headers: { 'Authorization': `Bearer ${token}` },
    });
    if (res.status === 401) {
      token = await generateApiJwt();
      res = await fetch(`${API_BASE_URL}/api/some-endpoint`, {
        headers: { 'Authorization': `Bearer ${token}` },
      });
    }

    // ❌ DON'T: Use direct fetch without JWT
    const res = await fetch(`${API_BASE_URL}/api/some-endpoint`);
    ```
  - **Exception**: Only use direct `fetch()` for proxy endpoints (e.g., `/api/proxy/...`) that are in the public routes list.
  - **Exception — raw `fetch` to backend base URL**: Rare (e.g. `ProfileBootstrapperApiServerActions`). If you must call `NEXT_PUBLIC_API_BASE_URL` with `fetch` instead of `fetchWithJwtRetry`, send **`Authorization: Bearer <token>`** and **`X-Tenant-ID: getTenantId()`** on every request. Service JWTs typically have **no** `tenant_id` claim; the backend `TenantContextFilter` / `getUserProfileByUserId` can return **400** (“Tenant context is required…”) if only query params like `tenantId.equals` are present. See **X-Tenant-ID for tenant-scoped backend APIs** below.
  - **Enforcement**: This rule applies to ALL server actions in the codebase. Any server action found using direct `fetch()` to backend APIs will be flagged for immediate correction.
  - **References:**
    - See `src/lib/proxyHandler.ts` for the `fetchWithJwtRetry` implementation
    - See `src/app/admin/events/[id]/media/ApiServerActions.ts` for correct usage
    - See `src/app/admin/ApiServerActions.ts` for correct usage

- **X-Tenant-ID for tenant-scoped backend APIs (user profile by user ID, etc.)**
  - **Problem**: Backend endpoints that resolve **user profile by Clerk user ID** (`/api/user-profiles/by-user/...` or criteria that map to `getUserProfileByUserId`) require **tenant context** on the HTTP request: **`X-Tenant-ID`** header (value from `getTenantId()`), or an equivalent tenant query/JWT claim **as implemented by your `TenantContextFilter`**. Sending only `tenantId.equals` in the query string may **not** set thread-local tenant for that code path → **HTTP 400** with a message like *“Tenant context is required to load a user profile by user id”*.
  - **DO — prefer shared helper**: Use **`fetchWithJwtRetry` from `@/lib/proxyHandler`**. It merges **`Authorization`** and **`X-Tenant-ID`** on every call (see `src/lib/proxyHandler.ts`). Use this for server actions and any code that calls the backend REST URL directly.
  - **DO — raw `fetch` when unavoidable**: Build headers with both `Authorization: Bearer <token>` and **`X-Tenant-ID: <getTenantId()>`** (plus `Content-Type` when needed). Example pattern: `src/components/ProfileBootstrapperApiServerActions.ts` (`tenantAuthHeaders`).
  - **DO — Pages Router proxies**: **`src/pages/api/proxy/user-profiles/[...slug].ts`** must forward using the **same** `fetchWithJwtRetry` from `@/lib/proxyHandler` (not a local copy that only sets `Authorization`). Local duplicate helpers have caused **by-user** proxy forwards to omit **`X-Tenant-ID`** and fail with 400.
  - **DON'T**:
    ```typescript
    // ❌ DON'T: Backend fetch with JWT only (missing X-Tenant-ID for tenant-scoped by-user APIs)
    await fetch(`${API_BASE_URL}/api/user-profiles/by-user/${userId}?tenantId.equals=${tenantId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    ```
  - **References**: `src/lib/proxyHandler.ts` (`fetchWithJwtRetry`), `src/components/ProfileBootstrapperApiServerActions.ts`, `src/pages/api/proxy/user-profiles/[...slug].ts`.

---
**Clerk Middleware Public Routes Configuration**
- **CRITICAL**: When using Clerk authentication with satellite domain support, the `publicRoutes` array in `src/middleware.ts` must include `/api/proxy(.*)` to allow public API data access.
- **Problem Solved**: Without this configuration, ALL `/api/proxy/` calls (even for public data like events, gallery, polls) will return 401 Unauthorized errors for unauthenticated users, breaking public pages.
- **Architecture Decision**:
  - **Clerk Middleware**: Handles user authentication and session management ONLY
  - **Backend API**: Handles data authorization via JWT authentication
  - **Public Data**: All `/api/proxy/` routes are public by default - backend determines what data requires authentication
- **Required Configuration** in `src/middleware.ts`:
  ```typescript
  import { authMiddleware } from "@clerk/nextjs/server";
  import { NextResponse } from "next/server";

  export default authMiddleware({
    // Define public routes that don't require authentication
    // IMPORTANT: Public API routes allow unauthenticated users to fetch public data
    publicRoutes: [
      '/',
      '/sign-in(.*)',
      '/sign-up(.*)',
      '/sso-callback(.*)',
      '/api/webhooks(.*)',
      '/api/public(.*)',
      '/api/proxy(.*)',  // Public API proxy routes for public data (events, etc.)
      '/mosc(.*)',
      '/events(.*)',
      '/gallery(.*)',
      '/about(.*)',
      '/contact(.*)',
      '/polls(.*)',
      '/charity-theme(.*)',
    ],

    // Satellite domain configuration for multi-domain support
    isSatellite: process.env.NEXT_PUBLIC_APP_URL?.includes('mosc-temp.com') || false,
    domain: process.env.NEXT_PUBLIC_APP_URL?.includes('mosc-temp.com') ? 'www.mosc-temp.com' : undefined,

    signInUrl: process.env.NEXT_PUBLIC_APP_URL?.includes('amplifyapp.com') || process.env.NEXT_PUBLIC_APP_URL?.includes('mosc-temp.com')
      ? 'https://www.adwiise.com/sign-in'
      : '/sign-in',

    afterAuth(auth, req) {
      const response = NextResponse.next();
      response.headers.set('x-pathname', req.nextUrl.pathname);
      return response;
    }
  });
  ```
- **Why This Works**:
  - Public pages (home, events, gallery, polls) can fetch data without requiring Clerk login
  - Admin/protected routes are still protected by backend JWT authentication
  - Separation of concerns: Clerk = user auth, Backend = data authorization
  - Satellite domain authentication flow remains intact
- **DO:**
  ```typescript
  // ✅ DO: Add /api/proxy(.*) to publicRoutes
  publicRoutes: [
    '/',
    '/api/proxy(.*)',  // Allows public data access
    // ... other public routes
  ],
  ```
- **DON'T:**
  ```typescript
  // ❌ DON'T: Omit /api/proxy(.*) from publicRoutes
  publicRoutes: [
    '/',
    '/sign-in(.*)',
    // Missing /api/proxy(.*) - will cause 401 errors on all public pages
  ],
  ```
- **Symptoms of Missing Configuration**:
  - Console errors: `GET /api/proxy/event-details 401 (Unauthorized)`
  - Public pages not loading data (events, gallery, polls)
  - Backend JWT authentication working but Clerk blocking all requests
- **Testing**:
  - Sign out completely and visit public pages (home, events, gallery)
  - Verify that data loads without authentication
  - Verify that admin pages still require login
- **References:**
  - See `src/middleware.ts` for complete implementation
  - See `src/app/layout.tsx` for satellite domain ClerkProvider configuration
  - See ByteRover memory for architectural decision details

---
**CRITICAL: Clerk Middleware ignoredRoutes Configuration for Mobile Browser Compatibility**
- **Problem**: Mobile browsers (especially iOS Safari, WhatsApp in-app browser, Chrome Mobile) can fail to make API proxy calls if Clerk middleware processes these requests, even when they're in `publicRoutes`.
- **Solution**: Use `ignoredRoutes` array in Clerk middleware to **completely bypass** authentication middleware for API proxy routes.
- **Rationale**:
  - `publicRoutes` tells Clerk "allow unauthenticated access but still run middleware"
  - `ignoredRoutes` tells Clerk "don't run middleware at all - completely skip"
  - Mobile browsers need the latter for reliable API proxy access
- **Required Configuration** in `src/middleware.ts`:
  ```typescript
  import { authMiddleware } from "@clerk/nextjs/server";

  export default authMiddleware({
    publicRoutes: [
      '/',
      '/events(.*)',
      '/api/proxy(.*)',  // Still needed for some Clerk features
      // ... other public routes
    ],

    // CRITICAL: Completely ignore API proxy routes (don't even apply Clerk middleware)
    ignoredRoutes: [
      // Ignore Next.js RSC prefetch requests
      '/(.*)?_rsc=(.*)$',
      // CRITICAL: Completely bypass Clerk for API routes
      '/api/webhooks/(.*)',
      '/api/proxy/(.*)',      // ← KEY for mobile browser compatibility
      '/api/stripe/(.*)',
      '/api/payment/(.*)',
      '/api/billing/(.*)',
      '/api/checkout/(.*)',
      '/api/diagnostic(.*)',
      '/api/logs(.*)',
    ],

    // ... rest of config
  });
  ```
- **DO:**
  ```typescript
  // ✅ DO: Add all API proxy routes to ignoredRoutes
  ignoredRoutes: [
    '/api/proxy/(.*)',
    '/api/webhooks/(.*)',
    '/api/stripe/(.*)',
  ]
  ```
- **DON'T:**
  ```typescript
  // ❌ DON'T: Rely only on publicRoutes for API proxy access
  publicRoutes: ['/api/proxy(.*)'],  // Not sufficient for mobile browsers
  ignoredRoutes: [],  // Missing - will cause mobile API failures
  ```
- **Symptoms of Missing ignoredRoutes**:
  - API calls work fine on desktop browsers
  - API calls fail on mobile browsers (iOS Safari, WhatsApp, Chrome Mobile)
  - Console shows "Failed to fetch" or CORS errors on mobile
  - Works in browser dev tools mobile emulation but fails on actual devices
- **Testing**:
  - Test on actual mobile devices (not just browser dev tools)
  - Test with WhatsApp in-app browser (common use case)
  - Test on both iOS Safari and Android Chrome
  - Verify API proxy calls succeed without authentication
- **References:**
  - Comparison with working `nextjs-template` project
  - Mobile browser debugging logs in CloudWatch
  - See `src/middleware.ts:63-75` for implementation

---
**Proxy Handler CORS Configuration Best Practices**
- **Problem**: Manually setting CORS headers in Next.js API routes (Pages Router) can cause conflicts and unexpected behavior, especially on mobile browsers.
- **Solution**: **DO NOT** set CORS headers manually in proxy handlers. Next.js handles CORS automatically for same-origin requests.
- **Rationale**:
  - Next.js API routes are same-origin by default (served from same domain as frontend)
  - Manual CORS headers can conflict with Next.js's built-in handling
  - Mobile browsers are more strict about CORS and can fail with manual headers
  - Adds unnecessary complexity and potential for misconfiguration
- **DO NOT:**
  ```typescript
  // ❌ DON'T: Manually set CORS headers in proxy handler
  export function createProxyHandler({ backendPath }: ProxyHandlerOptions) {
    return async function handler(req: NextApiRequest, res: NextApiResponse) {
      // WRONG: Don't set these manually
      res.setHeader('Access-Control-Allow-Origin', '*');
      res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
      res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

      // WRONG: Don't handle OPTIONS manually
      if (req.method === 'OPTIONS') {
        res.status(200).end();
        return;
      }

      // ... rest of handler
    };
  }
  ```
- **DO:**
  ```typescript
  // ✅ DO: Let Next.js handle CORS automatically
  export function createProxyHandler({ backendPath }: ProxyHandlerOptions) {
    return async function handler(req: NextApiRequest, res: NextApiResponse) {
      // No CORS headers needed - Next.js handles this
      try {
        const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
        // ... fetch backend API and return response
      } catch (err) {
        res.status(500).json({ error: 'Internal server error' });
      }
    };
  }
  ```
- **When You Might Need Manual CORS** (rare cases):
  - Cross-origin requests from a different domain (not same-origin)
  - Embedding your app in an iframe from another domain
  - Public API endpoints that need to be called from external websites
  - In these cases, use Next.js built-in CORS middleware, not manual headers
- **Exception - Next.js CORS Middleware** (if truly needed):
  ```typescript
  // Only if you need cross-origin access (rare)
  import Cors from 'cors';
  import { NextApiRequest, NextApiResponse } from 'next';

  const cors = Cors({
    methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
    origin: ['https://trusted-domain.com'],  // Specific domains, not '*'
  });

  function runMiddleware(req: NextApiRequest, res: NextApiResponse, fn: any) {
    return new Promise((resolve, reject) => {
      fn(req, res, (result: any) => {
        if (result instanceof Error) return reject(result);
        return resolve(result);
      });
    });
  }

  export default async function handler(req: NextApiRequest, res: NextApiResponse) {
    await runMiddleware(req, res, cors);
    // ... rest of handler
  }
  ```
- **Best Practices**:
  - Default to no manual CORS configuration
  - Next.js API routes are same-origin and don't need CORS
  - If you get CORS errors, first check:
    1. Is the request truly cross-origin? (different domain/port)
    2. Is Clerk middleware blocking the request? (use `ignoredRoutes`)
    3. Is the backend API returning CORS errors? (fix on backend)
  - Only add CORS configuration after confirming it's truly needed
  - Always use specific origins, never `'*'` in production
- **Mobile Browser Considerations**:
  - Mobile browsers (iOS Safari, Chrome Mobile) are stricter about CORS
  - Manual CORS headers can cause silent failures on mobile
  - Always test on actual mobile devices, not just dev tools emulation
  - Check browser console for CORS errors on mobile (use remote debugging)
- **References:**
  - Working `nextjs-template` project has no CORS headers in proxy handler
  - See `src/lib/proxyHandler.ts:17-19` for clean implementation
  - Next.js API Routes documentation: https://nextjs.org/docs/api-routes/introduction

---
**Mobile Browser API Call Debugging Checklist**
When API calls fail on mobile browsers but work on desktop:

1. **Check Clerk middleware configuration**:
   - ✅ Is `/api/proxy(.*)` in `ignoredRoutes`? (not just `publicRoutes`)
   - ✅ Are webhook routes in `ignoredRoutes`?
   - ✅ Check `src/middleware.ts` configuration

2. **Check CORS configuration**:
   - ✅ Remove manual CORS headers from proxy handlers
   - ✅ Let Next.js handle CORS automatically
   - ✅ Check if backend API has CORS issues

3. **Check client component patterns**:
   - ✅ Are you using `useParams()` correctly? (memoize with `useMemo`)
   - ✅ Are you preventing infinite fetch loops? (use `useRef` for fetch guards)
   - ✅ Are diagnostic/test fetches outside of data fetch useEffect?

4. **Check backend API**:
   - ✅ Is backend API accessible? (test with curl/Postman)
   - ✅ Are JWT credentials correct?
   - ✅ Is tenant ID configured correctly?
   - ✅ Check `NEXT_PUBLIC_API_BASE_URL` environment variable

5. **Mobile-specific testing**:
   - ✅ Test on actual devices (not just dev tools)
   - ✅ Test on iOS Safari, Android Chrome, and WhatsApp in-app browser
   - ✅ Use remote debugging to check console logs on mobile
   - ✅ Check CloudFront headers for mobile detection (if using AWS)

6. **Common mobile browser issues**:
   - `useParams()` proxy causing infinite re-renders
   - Clerk middleware blocking API routes (missing `ignoredRoutes`)
   - Manual CORS headers causing conflicts
   - Fetch loops from poorly managed useEffect dependencies
   - Session/token handling differences on mobile

7. **Quick fixes to try first**:
   ```typescript
   // Fix 1: Memoize useParams() result
   const eventId = useMemo(() => params?.id, [params?.id]);

   // Fix 2: Add fetch guard (prevent simultaneous fetches)
   const isFetchingRef = useRef(false);
   if (isFetchingRef.current) return;
   isFetchingRef.current = true;

   // Fix 3: Track fetched eventId (prevent re-fetching same ID on mobile re-hydration)
   const fetchedEventIdRef = useRef<string | string[] | null>(null);
   if (fetchedEventIdRef.current === eventId) return;
   // After fetch completes:
   fetchedEventIdRef.current = eventId;

   // Fix 4: Prevent premature "not found" message
   if (!data && !loading) return <NotFound />;  // Only show if done loading

   // Fix 5: Add ignoredRoutes to middleware
   ignoredRoutes: ['/api/proxy/(.*)']

   // Fix 6: Remove manual CORS headers from proxy handler
   ```

8. **Mobile Browser Re-hydration Issue**:
   - **Problem**: Mobile browsers (especially iOS Safari) can re-hydrate/re-mount components multiple times during navigation, causing duplicate fetches
   - **Symptoms**: Page flickers between "loading" and "not found", retries 100+ times, eventually loads
   - **Root Cause**: `useParams()` returns new proxy object on each render, `useMemo` dependency changes, useEffect re-runs
   - **Solution**: Track which eventId has been fetched using `useRef`, skip if already fetched
   - **Example**:
     ```typescript
     const fetchedEventIdRef = useRef<string | string[] | null>(null);

     useEffect(() => {
       async function fetchData() {
         // Prevent duplicate fetches
         if (isFetchingRef.current) return;
         // CRITICAL: Prevent re-fetching same ID (mobile re-hydration)
         if (fetchedEventIdRef.current === eventId) return;

         isFetchingRef.current = true;
         try {
           await fetch(...);
         } finally {
           isFetchingRef.current = false;
           fetchedEventIdRef.current = eventId; // Mark as fetched
         }
       }
       if (eventId) fetchData();
     }, [eventId]);
     ```

- **References:**
  - Mobile browser debugging: Chrome Remote Debugging, Safari Web Inspector
  - CloudWatch logs for AWS Amplify deployments
  - See working `nextjs-template` project for comparison
  - iOS Safari re-hydration: https://webkit.org/blog/7846/

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
