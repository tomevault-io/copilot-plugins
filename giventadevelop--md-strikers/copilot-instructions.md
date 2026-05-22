## common-app-router-aws-amplify-type-safety-best-practices

> - **Dynamic Route Params and Query Params (searchParams) Must Be Properly Unwrapped**

- **Dynamic Route Params and Query Params (searchParams) Must Be Properly Unwrapped**
  - In the Next.js App Router, `params` is a Promise in server components, but `searchParams` is passed directly as a prop.
  - **For dynamic route params in server components:**
    - Use: `const { id } = await params;`
    - Example:
      ```tsx
      export default async function Page({ params }: { params: Promise<{ id: string }> }) {
        const { id } = await params;
        // ...
      }
      ```
  - **For searchParams (query params) in server components:**
    - Destructure directly in the function signature:
      ```tsx
      export default async function Page({ searchParams }: { searchParams: { foo?: string } }) {
        const { foo } = searchParams;
        // ...
      }
      ```
    - **DO NOT** use `await params` or `use(params)` for `searchParams`.
    - **DO NOT** use `const { foo } = await searchParams;` or similar.
  - **For dynamic route params in client components:**
    - Use: `const { id } = use(params);`
    - Example:
      ```tsx
      import { use } from "react";
      export default function Page({ params }: { params: Promise<{ id: string }> }) {
        const { id } = use(params);
        // ...
      }
      ```
  - **Anti-patterns (DON'T):**
    - `const { id } = params;` // ❌ params is a Promise in server components
    - `const { foo } = await searchParams;` // ❌ searchParams is not a Promise
    - `const { searchParams } = await params;` // ❌ not needed for searchParams
    - `const { id } = use(params);` in server components // ❌ use() is for client only
  - See: https://nextjs.org/docs/messages/sync-dynamic-apis
  - **Rationale:** Prevents runtime errors and ensures forward compatibility with future Next.js versions.

- **Public Pages and Middleware (publicPaths)**
  - For any page that should be accessible without authentication (such as event success/confirmation pages), ensure its route is included in the `publicPaths` array in `src/middleware.ts`.
  - Example:
    ```typescript
    const publicPaths = [
      '/',
      '/sign-in(.*)',
      '/sign-up(.*)',
      '/event(.*)',
      '/event/success(.*)', // Explicitly make event success page public
      // ...other public routes
    ];
    ```
  - **DO:** Add `/event/success(.*)` to `publicPaths` to make the event success page public.
  - **DON'T:** Omit a public page from `publicPaths` if it should be accessible without login—this will cause 401 errors for unauthenticated users.
  - **Rationale:** Clerk (or other auth middleware) will block access to any route not listed in `publicPaths` for unauthenticated users. Always update this list when adding new public pages.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
