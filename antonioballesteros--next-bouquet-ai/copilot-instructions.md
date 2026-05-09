## style-guide

> Project style guide for Next.js (React, TypeScript, Tailwind, shadcn)


# Style guide — next-bouquet-ai

## Stack and conventions

- **Next.js 16** with App Router. Routes live under `app/`.
- **React 19**: functional components and hooks only.
- **TypeScript**: strict typing; avoid `any`. Define interfaces/types for props and data.
- **Tailwind v4**: use Tailwind classes for styling; avoid CSS-in-JS or inline styles unless necessary.
- **shadcn/ui**: use shadcn components (from `components/ui/`) for buttons, forms, dialogs, etc.; customise via Tailwind when needed. Prefer adding components with `npx shadcn@latest add` rather than building from scratch.
- **Supabase**: DB (PostgreSQL), Auth, Storage (e.g. images).
- **Drizzle**: ORM for queries to Supabase/Postgres.
- **Tanstack Query**: Client-side data fetching (cache, mutations); requests go to server actions or API routes, not directly to Supabase from the browser.
- **Auth**: Supabase Auth; use `@supabase/ssr` for session in Next (server and middleware).
- **Image creation**: Vercel AI + Replicate, model `black-forest-labs/flux-schnell`; run only on the server (server actions or API routes), never expose keys to the client.

### Stack usage and structure

- **Drizzle**: Put schema and migrations in a dedicated folder (e.g. `lib/db/` or `db/`). Single DB client built from Supabase connection; use it in Server Components, server actions, and API route handlers.
- **Tanstack Query**: Wrap the app with `QueryClientProvider`. Put query/mutation logic in custom hooks (e.g. `useImages()`, `useCreateImage()`). Hooks call server actions or `fetch` to app API routes; do not call Supabase or Replicate from the client.
- **Supabase client**: Use a **server** Supabase client (e.g. `createServerClient` from `@supabase/ssr`) in server code and middleware; use a **browser** client only in `'use client'` components when you need real-time or client-side auth (e.g. sign in/out).
- **AI / Replicate**: Keep all Replicate and Vercel AI calls in server-only code (server actions or `app/api/.../route.ts`). Never send API keys or call Replicate from the client.
- **Storage (Supabase)**: Upload/signed URL logic on the server; client receives only URLs or signed URLs returned from the server.
- **shadcn/ui**: Components live under `components/ui/`. Use existing shadcn primitives (Button, Card, Dialog, Input, etc.) for consistency and accessibility; extend or compose them rather than duplicating. Keep the props-interface rule for any wrapper or custom components you add.
- **Middleware vs proxy**: Do **not** use `middleware.ts`. Whenever request interception, rewrites, or similar logic is needed, use `proxy.ts` instead.

## Structure and naming

- Components: PascalCase (`MyComponent.tsx`).
- Hooks and utilities: camelCase (`useSomething.ts`, `formatDate.ts`).
- Pages and routes: lowercase/kebab in URLs; folders under `app/` follow the desired route.
- Constants and types: UPPER_SNAKE for constants, PascalCase for types/interfaces.

## Components and files

- One main component per file; small related components may live in the same file when it makes sense.
- Default export for pages (`page.tsx`); named exports for reusable components.
- Use `'use client'` only where needed (interactivity, browser hooks); keep the rest as Server Components by default.

### Component props interface

- For every component, define a **dedicated props interface** named after the component plus `Props`: `Card` → `CardProps`, `UserAvatar` → `UserAvatarProps`.
- **Never** define the props type inline in the component signature (e.g. avoid `function Card({ title }: { title: string })`).
- Declare the interface **above** the component; this keeps the component signature clear and the props contract visible in one place.

## Code

- Double quotes for strings in TS/TSX unless the linter dictates otherwise.
- Prefer `async/await` over `.then()`.
- Keep functions and components short; extract logic to hooks or utilities when they exceed ~30 lines.
- Comments in English when needed.

### Functions: named declarations vs arrow functions

- **Components and top-level logic**: use **named function declarations** (`function Name() {}`). Better stack traces, matches Next.js docs, hoisting when needed.
- **Event handlers and callbacks**: use **arrow functions** (`const handleClick = () => {}` or inline `() => {}`).

```tsx
// ✅ Components and exported logic
export default function Page() { ... }
export function Card({ title }: CardProps) { ... }

// ✅ Handlers and callbacks
const handleSubmit = () => { ... };
items.map((item) => <Item key={item.id} {...item} />);
```

## Example component

(Follows the props-interface rule above.)

```tsx
// Presentational component with clear typing
interface CardProps {
  title: string;
  children: React.ReactNode;
}

export function Card({ title, children }: CardProps) {
  return (
    <article className="rounded-lg border p-4">
      <h2 className="text-lg font-semibold">{title}</h2>
      {children}
    </article>
  );
}
```

---
> Source: [antonioballesteros/next-bouquet-ai](https://github.com/antonioballesteros/next-bouquet-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
