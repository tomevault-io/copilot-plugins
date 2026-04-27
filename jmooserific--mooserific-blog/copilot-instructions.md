## mooserific-blog

> Family photo blog built with Next.js (App Router), TypeScript, and Tailwind CSS. Uses Cloudflare D1 for metadata and Cloudflare R2 for media storage. Deployed on Vercel.

# CLAUDE.md

## Project Overview

Family photo blog built with Next.js (App Router), TypeScript, and Tailwind CSS. Uses Cloudflare D1 for metadata and Cloudflare R2 for media storage. Deployed on Vercel.

## Code Style & TypeScript

- **Strict TypeScript.** No `any` types. Use `unknown` + type guards when the type is genuinely unknown. Prefer interfaces for object shapes, type aliases for unions/intersections.
- Prefer `const` over `let`. Never use `var`.
- Use named exports, not default exports (except for Next.js page/layout conventions that require them).
- Colocate types with the code that uses them. Only extract to a shared `types/` directory when a type is used across 3+ files.
- Use discriminated unions over optional fields when modeling variants (e.g., `{ status: 'loading' } | { status: 'error'; error: string } | { status: 'success'; data: T }`).

## Next.js Conventions

- **App Router only.** No `pages/` directory patterns.
- Default to **Server Components**. Only add `'use client'` when you need interactivity, browser APIs, or hooks. Keep client components as leaf nodes — push them down the tree.
- Use `loading.tsx` and `error.tsx` for route-level Suspense/error boundaries.
- Use Next.js `<Image>` for all images. Always provide explicit `width`/`height` or use `fill` with a sized container. Set appropriate `sizes` attribute for responsive images.
- Data fetching belongs in Server Components or Route Handlers (`app/api/`). Never fetch in `'use client'` components unless it's a user-initiated action (e.g., form submission).
- Use `next/navigation` (`useRouter`, `usePathname`, `useSearchParams`), never `next/router`.
- Metadata: use the `generateMetadata` function or static `metadata` export for SEO, not `<Head>`.

## Tailwind CSS

- Use Tailwind utility classes directly. No CSS modules, no `@apply` unless extracting a genuinely reusable component pattern.
- Prefer responsive Tailwind variants (`sm:`, `md:`, `lg:`) over custom media queries.
- Use Tailwind's color palette and spacing scale consistently. Don't use arbitrary values (`[13px]`) unless the design literally requires a non-standard value.
- For dark mode, use the `dark:` variant if/when added.

## Component Patterns

- One component per file. File name matches the component name in PascalCase.
- Props interfaces go in the same file, named `{ComponentName}Props`.
- Keep components small and focused. If a component exceeds ~150 lines, break it up.
- Prefer composition over configuration — pass children/render props rather than building mega-components with dozens of boolean flags.
- Use semantic HTML (`<article>`, `<nav>`, `<figure>`, `<figcaption>`, `<time>`, `<main>`, `<section>`). This is a photo blog — `<figure>` and `<figcaption>` are your friends.

## Performance

- Minimize client-side JavaScript. Every `'use client'` component adds to the bundle.
- Use dynamic `import()` with `next/dynamic` for heavy client components that aren't above the fold.
- Use `loading="lazy"` behavior via Next.js `<Image>` for below-fold images. Use `priority` for the LCP image.
- Avoid `useEffect` for data that can be fetched on the server.
- Prefer `Map` and `Set` over repeated array `.find()`/`.includes()` when operating on large collections.

## Media Storage

- Always store the original uploaded file in R2 alongside generated variants. The `PhotoAsset` type tracks both the optimized base URL and the original via `originalUrl`/`originalContentType`.
- Never discard or skip uploading the original — we need it to re-optimize or recover from processing bugs.

## Error Handling & Security

- Validate all external input (URL params, form data, API request bodies) at the boundary using Zod schemas. Never trust client input.
- Use parameterized queries for D1. Never interpolate user input into SQL strings.
- Sanitize any user-generated content before rendering. Be cautious with `dangerouslySetInnerHTML`.
- Return proper HTTP status codes from Route Handlers (400 for validation errors, 404 for missing resources, 500 for unexpected failures).
- Never expose internal error details to the client in production. Log the full error server-side, return a generic message to the client.
- Keep secrets in environment variables. Never commit them. Use `.env.local` for development.

## Testing & Quality

- When writing or modifying a function with non-trivial logic, include or update the corresponding test.
- Prefer small, pure, testable functions extracted from components over testing component rendering.
- Name test files `*.test.ts(x)` colocated with the source file.

## Git Practices

- Write clear, imperative commit messages: "Add album pagination" not "added stuff".
- Keep commits focused on a single logical change.
- Don't commit generated files, `node_modules`, `.env.local`, or `.next/`.

## When In Doubt

- Readability wins over cleverness. Write code that's obvious at a glance.
- Fewer abstractions are better than premature abstractions. Duplicate twice before extracting.
- If a decision is ambiguous, leave a brief `// TODO:` comment explaining the trade-off rather than silently picking one path.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmooserific) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
