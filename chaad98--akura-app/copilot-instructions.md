## akura-app

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

## Project Instructions for Claude

Frontend for **Akura**: small Malaysian renovation contractors and their clients share
one timestamped record of approvals and work progress, replacing scattered WhatsApp
chats.

Two roles use the same pages with different affordances. The **contractor** logs work
progress and raises variation orders; the **client** approves or declines them.

## Stack

- Next.js (App Router) + TypeScript
- **Tailwind CSS v4** — CSS-first config, see below
- Auth: **httpOnly cookies** — not localStorage, not `Authorization: Bearer`

## Repo boundaries

The backend is a **separate repository** (`akura-core`, NestJS). Never create backend
folders, database code, or Next.js API routes that duplicate it.

**No database access in this repo.** No Prisma, no `DATABASE_URL`, no direct SQL. All
data comes from `api.akura.my` over HTTP. The backend owns the domain rules,
authorization, and logging — talking to Postgres from here would bypass all of them.

## Documents

| File                    | What it is                                             |
| ----------------------- | ------------------------------------------------------ |
| `docs/FRONTEND_SPEC.md` | Rendering strategy, pages, API contract, SEO, security |
| `docs/BACKEND_SPEC.md`  | Reference copy — §6 endpoints, §7 auth/cookies         |
| `docs/BUILD_PLAN.md`    | Phase order and exit criteria                          |

Work in build-plan phase order. Match the backend spec's routes exactly — never invent
or guess route shapes.

---

## Rendering strategy — the central decision

Akura has two halves with opposite requirements.

| Half       | Pages                                   | Rendering                                         |
| ---------- | --------------------------------------- | ------------------------------------------------- |
| **Public** | `/`, `/pricing`                         | **SSG** — Server Components, static at build time |
| **App**    | `/login`, `/dashboard`, `/project/[id]` | **CSR** — client components fetching from the API |

**Do not server-render authenticated pages.** SSR there means browser → Vercel →
`api.akura.my` → back, adding a hop per navigation across possibly different regions.
SSR's benefits — SEO and content in first paint — don't apply to a page only one
logged-in person sees. CSR with a loading state is simpler and usually faster here.

Server Components are still right for static layout shells and the public pages; they
ship zero JS. Just don't reach for server-side data fetching on authenticated views.

## SEO — public pages only

Authenticated pages must **never** be indexed. `robots.ts` disallows `/dashboard`,
`/project/`, `/login`. `sitemap.ts` lists public pages only.

Public pages get: per-page `metadata` with OG and Twitter tags, `metadataBase`, JSON-LD
`SoftwareApplication`, `next/font` and `next/image` (both prevent layout shift, a
ranking factor), one `<h1>` per page, real heading hierarchy.

Write copy for a Malaysian renovation contractor tired of arguing with clients over
WhatsApp — not for generic "project management".

## Tailwind v4 — setup differs from older guides

Anything referencing `tailwind.config.js` or `@tailwind base;` is v3-era and does not
apply.

- **No `tailwind.config.ts`.** Content paths auto-detected. Do not create one.
- `globals.css` starts with `@import "tailwindcss";` — not the three `@tailwind`
  directives.
- Theme customisation in CSS via `@theme`, not a JS config.
- Next.js still needs a thin `postcss.config.mjs` with `@tailwindcss/postcss`.
  **Do not add `autoprefixer`** — v4 prefixes internally; keeping it double-prefixes.

---

## Rules that are easy to get wrong

**Auth is cookie-based.** There is no token in JavaScript. Never read or write
`localStorage` or `sessionStorage`, never set an `Authorization` header, never expect
`accessToken` in a response body — if you reach for it, the backend is misconfigured.

**Every request needs `credentials: 'include'`.** The most common failure mode:
without it the cookie is never sent and authenticated calls 401 while the code looks
correct.

**`GET /auth/me` is the only way to know who is logged in.** The token can't be decoded
client-side. An `AuthContext` calls it once on mount and exposes `user`, `loading`,
`refresh()`, `logout()`. Render a loading state during that first call or protected
pages flash their empty state.

**Logout is a server call** — the client cannot delete an httpOnly cookie.

**`middleware.ts` route protection is UX, not security.** It checks cookie _presence_,
not validity, and cannot verify a signature. The backend is the actual gate. The same
goes for role-based rendering: hide controls a role can't use, but assume anyone can
call the API directly.

**Money is integer cents on the wire.** Sending `Math.round(Number(rm) * 100)`,
displaying `(costCents / 100).toFixed(2)`. Get this wrong and RM 3,200 bills as RM 3.20.

**`decideApproval` hits `PATCH /approvals/:approvalId`** — flat, not nested under a
project. Everything else is nested.

**Never render raw HTML from API data.** No `dangerouslySetInnerHTML` — project names
and VO descriptions are user input. React escapes by default; don't opt out.

**No secrets in this repo.** Anything `NEXT_PUBLIC_` is visible in the browser bundle.
Only `NEXT_PUBLIC_API_URL` belongs there.

---

## Type safety

**TypeScript only.** No `.js` / `.jsx`. Rename the generated `next.config.js` to
`next.config.ts` and type it with `NextConfig`. Unavoidable exceptions:
`package.json`, `tsconfig.json`, `.eslintrc.json`, `postcss.config.mjs` (PostCSS has no
TS config support), generated `next-env.d.ts`.

**`any` is banned** outside `*.test.ts` / `*.test.tsx`. Enforce with `strict: true`,
`noImplicitAny`, `noUncheckedIndexedAccess`, plus ESLint
`@typescript-eslint/no-explicit-any` with test overrides. Use `unknown` plus narrowing
where a shape genuinely isn't known.

**`lib/types.ts` declares the API response types.** No shared package exists, so this
repo mirrors the backend's shapes by hand: `User`, `Project`, `ProjectDetail`, `Stage`,
`Approval`, `Paginated<T>`, and the `Role` / `StageStatus` / `ApprovalStatus` unions.
When a backend response changes, update this file in the same sitting — a stale type
compiles fine and fails at runtime.

Dates arrive as ISO strings, not `Date`. Type them `string`, convert at display.

**Make the fetch wrapper generic** so `res.json()`'s `any` never escapes:
`request<T>(path, options): Promise<T>`. One narrowing point, every caller typed free.

Where `any` sneaks in: `useState<any>(null)` → `useState<ProjectDetail | null>(null)`;
event handlers → `React.FormEvent<HTMLFormElement>`; caught errors → `unknown`, narrow
with `instanceof Error`; select values → a type guard rather than `as Role`.

---

## API client

One module (`lib/api.ts`) wrapping `fetch`. No scattered `fetch` calls in pages. Throw
a real `Error` on non-2xx using the backend's `message` so validation errors reach the
user instead of the console.

## Pages

| Route           | Rendering | Purpose                                           |
| --------------- | --------- | ------------------------------------------------- |
| `/`             | SSG       | Landing                                           |
| `/pricing`      | SSG       | Optional for v1                                   |
| `/login`        | CSR       | Register or log in, choosing contractor or client |
| `/dashboard`    | CSR       | Project list; create form (contractor only)       |
| `/project/[id]` | CSR       | Shared view: stage checklist + approvals          |

App pages need `'use client'`; public pages stay Server Components.

Approve/decline buttons show only while an approval is `PENDING` and disappear once
decided — a settled decision must not look re-clickable.

After any mutation, re-fetch. Simple and always correct at this scale.

---

## Out of scope for v1

Photo upload UI, a component library or design system, optimistic updates, refresh
token handling, dark mode, i18n. Style with Tailwind utilities directly — don't build
abstractions over them yet.

## Testing

Use a normal window plus an incognito window to be contractor and client at once.
Cookies are shared across tabs of one profile, so two tabs would both be logged in as
whoever authenticated last.

---
> Source: [chaad98/akura-app](https://github.com/chaad98/akura-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
