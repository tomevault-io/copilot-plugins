## nhungskycity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Features implemented

Real-estate lead-gen landing page for Sunshine Sky City (Quận 7, HCMC), plus an internal admin module. Status as of the latest commit:

- **Public marketing site** (`src/app/page.tsx`): `Hero` → `ProjectInfo` → `Location` → `Amenities` → `GoldValues` → `ApartmentDesign` → `FloorPlan` → `Pricing`, with a scrollspy `Header`, persistent `Footer`, and floating call/Zalo buttons (`FloatingActions`).
- **Entry popup** (`EntryPopup.tsx`): Radix `Dialog` that auto-opens immediately on mount, gated by `sessionStorage["hasSeenEntryPopup"]` so it only shows once per browser session; embeds the same `LeadForm` (`source="Popup Chào Mừng"`).
- **Lead capture pipeline**: `LeadForm` (react-hook-form + Zod) → `POST /api/leads` → Supabase insert → best-effort Telegram notification → best-effort Meta CAPI `Lead` event → toast feedback. See "Architecture" below for the full data flow.
- **SEO**: full `Metadata` export in `src/app/layout.tsx` (title template, keywords, OpenGraph image, Twitter card, canonical, robots) sourced from `siteConfig.seo`.
- **`/cms` admin module**: Basic Auth-gated (`CMS_USERNAME`/`CMS_PASSWORD`) dashboard with a leads table + Excel export, and a settings form to edit the Meta Pixel ID / CAPI token stored in Supabase. See "CMS module" below.
- **Meta Pixel + Conversions API**: pixel script injected site-wide when configured; CAPI `Lead` event fired server-side with hashed PII after every successful lead insert. See "Meta Pixel + Conversions API" below.
- **Verified end-to-end against a real Supabase project**: form submission → row visible in `leads` table → visible in `/cms` (this surfaced and fixed the fetch-caching bug documented below).
- **Cloned from the `sunshine-sky-city` boilerplate; mounted at `nhungtranreal.site/skycity`** via Next.js `basePath` on its own standalone domain — no gateway/rewrite project involved (unlike the base boilerplate). See "Domain architecture" below — this is the one change most likely to break silently if a future edit adds a raw local image `src` or a bare internal `fetch()` call.
- **White-label boilerplate refactor applied**: `src/config/site.ts` centralizes `layoutOrder` (section render order, consumed by `src/app/page.tsx`'s `sectionComponents` registry) and `images` (every local image path, grouped by section). A future clone only ever needs to touch domain/phone/`layoutOrder`/`images` in that one file plus the files under `public/images/`, never the `.tsx` components. `scripts/clone-project.mjs` (inherited from the base boilerplate) automates copying the repo to a new sibling project.

## Project rules (immutable)

These rules were set at project inception and must be preserved across all future changes:

1. **No placeholder code.** Never write `// add code here`, `// similar to above`, or half-finished implementations. Every file must be complete, working code — regardless of length.
2. **Every form/input must validate with Zod** before submission (see `src/lib/validations.ts`, enforced client-side via `@hookform/resolvers/zod` and again server-side in the API route).
3. **Every API fetch must have `try/catch` with a UI fallback** for network/API failure — never let a request fail silently. See `src/app/api/leads/route.ts` and `LeadForm.tsx`'s error toast as the reference pattern.
4. **All contact info (hotline, Zalo, address, email, nav links) lives only in `src/config/site.ts`.** Never hardcode phone numbers or links inside components — import from `siteConfig`.
5. **Mobile-first, fully responsive.** No layout may break or images distort on phone-sized viewports; CTA buttons must stay large/tappable.
6. **Images live in `public/images/` and are referenced as local paths** (`/images/xxx.jpg`), not external URLs. Prefer real project renders over generic stock/placeholder photos when available — see "Image sourcing" below.
7. **A global `<ErrorBoundary>` wraps the app** (in `src/app/layout.tsx`) — do not remove it or render pages outside it.
8. **Follow standard Next.js App Router file organization**: `src/app`, `src/components`, `src/components/ui` (Radix wrappers), `src/lib`, `src/config`.

### Tech stack constraints

- **Next.js 14 (App Router) + TypeScript strict.** Do not downgrade strictness or eject from the App Router.
- **Tailwind CSS v3 only.** Do not introduce MUI, Ant Design, Chakra UI, or any other component/styling library.
- **Radix UI primitives** (`@radix-ui/react-tabs`, `@radix-ui/react-dialog`, `@radix-ui/react-slider`) wrapped under `src/components/ui/` — build new interactive primitives the same way rather than reaching for a third-party UI kit.
- **lucide-react** for all icons.
- **react-hook-form + Zod** for all forms (see rule 2).
- **Supabase (Postgres)** as the lead-capture datastore; schema lives in `supabase/schema.sql` and must be kept in sync with any change to the `leads` table shape.
- **Two Supabase clients, never cross them**: `src/lib/supabase.ts` (`getSupabaseClient`) uses the anon key and is the only client allowed in public-facing code (`/api/leads`'s insert). `src/lib/supabase-admin.ts` (`getSupabaseAdminClient`) uses `SUPABASE_SERVICE_ROLE_KEY`, bypasses RLS, and must only be imported from server-only code behind the `/cms` auth gate or from route handlers (`/api/settings`, the CAPI lookup in `/api/leads`, and the root layout's pixel-id fetch) — never from a `"use client"` file.

## Commands

```bash
npm install       # install dependencies
npm run dev       # start dev server (localhost:3000)
npm run build     # production build — also runs TypeScript + ESLint checks; treat build failures as blocking
npm run start     # run the production build
npm run lint      # ESLint only
```

There is no test suite configured in this project.

When testing UI changes visually, kill any stale dev server / `.next` build first if you see a "Cannot find module .../vendor-chunks" error — this happens when a `next build` output and `next dev` output collide in the same `.next` directory:
```bash
lsof -ti:3000 -sTCP:LISTEN | xargs -r kill
rm -rf .next
npm run dev
```

## Architecture

**Data flow for lead capture** (the core piece of business logic in this app):
`LeadForm.tsx` (react-hook-form + zodResolver using the shared `leadFormSchema`) → `POST /api/leads` (`src/app/api/leads/route.ts`, re-validates with the same Zod schema) → insert into Supabase `leads` table (`src/lib/supabase.ts`) → best-effort Telegram notification (`src/lib/telegram.ts`) → best-effort Meta Conversions API `Lead` event (`src/lib/meta-capi.ts`, see "Meta Pixel + Conversions API" below) → JSON `{ success, message }` back to the client → toast success/error (`src/components/ui/toast.tsx`). The Telegram and CAPI steps are each independently try/caught and only logged on failure — neither can fail a request after the Supabase insert has already succeeded.

`LeadForm` is a single reusable component parameterized by a `source` string (e.g. `"Section Giá bán & Thanh toán"`) so every placement on the page reports where the lead came from without duplicating form logic.

**Page composition** (`src/app/page.tsx`): sections are plain components rendered in sequence, each wrapping itself in a `<section id="...">` that matches an entry in `siteConfig.nav`. The order is: `Hero` → `ProjectInfo` → `Location` → `Amenities` → `GoldValues` → `ApartmentDesign` → `FloorPlan` → `Pricing`, followed by a persistent `Footer` and `FloatingActions` (call/Zalo buttons) outside `<main>`.

**Header scrollspy** (`src/components/Header.tsx`): the nav highlight is driven purely by `siteConfig.nav` — it builds an `IntersectionObserver` over `document.querySelector(item.href)` for every nav entry. Any new top-level section that should appear in the nav must (a) have a matching `id` on its `<section>` and (b) be added to `siteConfig.nav`; sections without a nav entry (e.g. `GoldValues`'s `#gia-tri-vang`) render fine but won't be scrollspy-highlighted or get a header link.

**JSX text vs. string literals — HTML entity gotcha**: Babel/TypeScript's JSX transform decodes named HTML entities (`&amp;`, `&mdash;`, `&ldquo;`, etc.) only inside literal JSX text children. The same entity inside a plain JS string (e.g. a `description: "..."` field in a data array later interpolated via `{item.description}`) is **not** decoded and will render literally as `&mdash;`. Use real Unicode characters (`—`, `–`, `"`) in string constants/arrays; either form works in direct JSX text, but prefer `&ldquo;`/`&rdquo;` there too since ESLint's `react/no-unescaped-entities` rule rejects raw `"` typed directly as JSX text.

**Image sourcing**: renders in `public/images/` were sourced from the developer's own public marketing sites (skycity.sunshinegroup.vn, sunshineskycity.vn) rather than generic stock photos, to keep the landing page visually consistent with the real project. When adding more images, prefer that same source before falling back to Unsplash placeholders, and keep filenames descriptive of content (`amenity-*`, `interior-*`, `hero-*`, etc.) rather than the numeric names they're downloaded with.

## Environment variables

Defined in `.env.example`, required at runtime for the lead form to actually persist/notify (without them the API route degrades to a logged warning + 502, which the client surfaces as a friendly error toast — this is expected fallback behavior, not a bug):

- `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`
- `SUPABASE_SERVICE_ROLE_KEY` — required for `/cms`, `/api/settings`, the Meta CAPI lookup in `/api/leads`, and the Meta Pixel fetch in the root layout. Missing this degrades each of those the same way as above (logged error + graceful fallback: CMS shows a red banner instead of crashing, pixel script is simply omitted).
- `CMS_PASSWORD` (required) and `CMS_USERNAME` (optional, defaults to `admin`) — Basic Auth credentials checked by `src/middleware.ts` to allow **any** access to `/cms` or `/api/settings`. If `CMS_PASSWORD` is unset, the middleware hard-denies (401) rather than fail open.

`supabase/schema.sql` creates the `leads` table with RLS enabled and a policy allowing only anonymous **inserts** (no read/update/delete from the client key). `supabase/settings-schema.sql` creates the singleton `site_settings` table (`id` fixed to `1` via a check constraint) with RLS enabled and **no policies at all** — it is intentionally unreachable by the anon/authenticated roles; only the service role key can touch it.

## CMS module (`/cms`)

A single internal admin page, not part of the public marketing site, gated by Basic Auth (`src/middleware.ts`, matcher `/cms/:path*` and `/api/settings/:path*`) checked against `CMS_USERNAME`/`CMS_PASSWORD`. There is no session/cookie — every request re-checks the `Authorization` header.

`src/app/cms/page.tsx` is an async Server Component with **both** `export const dynamic = "force-dynamic"` **and** `export const fetchCache = "force-no-store"`. Both are required: `dynamic` alone only controls the route's rendering mode, it does *not* stop Next.js's fetch Data Cache from caching the plain GET requests `@supabase/supabase-js` issues internally — this was a real bug found via end-to-end testing (a lead saved fine but `/cms` kept showing a stale list until `fetchCache` was added). It fetches leads and `site_settings` in parallel via `getSupabaseAdminClient()`, each wrapped in its own try/catch that degrades to an empty/`null` result plus a visible red banner (never a crash) if Supabase is unreachable. Two Radix `Tabs` panels:

- **Danh sách Leads**: `LeadsTable` (plain server-rendered table, no client JS needed) + `ExportExcelButton` (`"use client"`, builds an `.xlsx` with the `xlsx` package client-side from the `leads` prop already fetched server-side — it does not call any API).
- **Cấu hình Tracking**: `SettingsForm` (`"use client"`, react-hook-form + `siteSettingsSchema`) posts to `POST /api/settings`, then calls `router.refresh()` on success so the Server Component re-fetches and reflects the saved values — there is no client-side state mirroring of settings beyond the form's own fields.

**`xlsx` package note**: the npm-registry build (0.18.5) carries two known high-severity advisories (prototype pollution, ReDoS), both in the *parsing* (`XLSX.read`/`readFile`) code path. This app only ever calls the *write* path (`json_to_sheet` / `writeFile`) on data it already trusts (leads from its own database) and never parses untrusted spreadsheet input, so the exploitable path is not reachable here. SheetJS's own patched builds are distributed outside the npm registry (cdn.sheetjs.com) — if that trade-off ever needs revisiting, that's the upgrade path, not `npm audit fix --force` (which has no fixed version to offer).

## Meta Pixel + Conversions API (CAPI)

Both the pixel ID and the CAPI access token are stored in `site_settings`, editable only via `/cms`, never in `siteConfig` or `.env` — this is the one piece of "site configuration" that intentionally lives in the database instead of rule 4's `src/config/site.ts`, because it must be editable at runtime without a redeploy.

- **Frontend pixel** (`src/app/layout.tsx`): `RootLayout` is an async Server Component that calls `getSupabaseAdminClient()` to read `meta_pixel_id` before rendering. If present, it injects the standard fbevents.js bootstrap via `next/script` (`strategy="afterInteractive"`) plus a `<noscript>` pixel `<img>` fallback; if the fetch fails or no ID is configured, both are simply omitted (logged, not thrown) — the marketing site must never break because of this. Note this makes every route inherit a per-request Supabase read at render time (no caching layer), which is an intentional trade-off for "editable without redeploy," not an oversight.
- **Backend CAPI** (`src/app/api/leads/route.ts`, Action 3, after the Supabase insert and the Telegram notification): looks up `site_settings` via the admin client, and only if **both** `meta_pixel_id` and `capi_token` are present calls `sendMetaLeadEvent` (`src/lib/meta-capi.ts`), which hashes `phone`/`email` with SHA-256 (Meta requires PII fields pre-hashed, never sent in the clear) and POSTs a `Lead` event to `graph.facebook.com`. Wrapped in its own try/catch that only logs on failure — exactly like the Telegram step, a CAPI failure must never fail the lead submission the user is waiting on.

## Deployment notes (Vercel)

This project is deployed on Vercel from the `main` branch. Two recurring failure modes hit repeatedly during development, both worth checking first whenever "it works locally but not on the live site" comes up:

1. **`.env.local` is gitignored and never reaches Vercel.** Every env var in `.env.example` must be added separately in Vercel → Project → Settings → Environment Variables (Production, and Preview if used). Forgetting this makes `/api/leads` return 502 (`getSupabaseClient` throws) and makes `/cms` hard-401 (`CMS_PASSWORD` unset) — both are the *correct* fail-safe behavior of this codebase, not bugs, but they look identical to a real outage from the browser console.
2. **Env var changes on Vercel don't apply to already-built deployments.** After adding/editing variables, you must trigger a new deployment (Deployments → latest → "..." → Redeploy, or just push a commit) for them to take effect.

When debugging a "form/login doesn't work" report, first ask whether the user is testing the deployed domain or `npm run dev` locally — the two have completely independent env var configuration.

## Domain architecture: nhungtranreal.site (standalone domain + basePath, no gateway)

Unlike the base `sunshine-sky-city` boilerplate this was cloned from, this project does **not** run behind the `canhoquan7-gateway` rewrite project. It owns its own domain, `nhungtranreal.site`, assigned directly to this app's own Vercel project. There is no separate gateway repo involved and nothing to add there.

The site still lives at `/skycity` rather than the domain root (`next.config.mjs` sets `basePath: "/skycity"`), purely as a URL-structure choice for this project, not for multi-tenant routing. Consequences of `basePath`, all already applied — same mechanics as the base boilerplate, just a different domain and prefix:

- `next/image` does **not** auto-prefix local `src` values (documented Next.js exception, unlike `next/link`). Every local image reference goes through `asset()` (`src/config/site.ts`) instead of a raw string — e.g. `src={asset(siteConfig.images.hero)}`. `asset()` passes external URLs (Unsplash placeholders) through unchanged via an `http(s)://` check. **Any new local image reference must use `asset()`, or it will 500 in dev / 404 in prod.**
- Client-side `fetch()` calls to this app's own API routes are **not** basePath-aware either — `LeadForm.tsx` and `SettingsForm.tsx` build the URL as `` `${siteConfig.basePath}/api/leads` `` / `/api/settings` rather than a bare string. Any new client-side fetch to an internal route must follow the same pattern.
- `src/middleware.ts`'s matcher (`/cms/:path*`, `/api/settings/:path*`) does **not** need manual prefixing — Next.js applies basePath to middleware matchers automatically.
- `siteConfig.url` is the full external URL (`https://nhungtranreal.site/skycity`, no trailing slash) used for canonical/OpenGraph. `siteConfig.seo.ogImage` is a **full absolute URL**, not a root-relative path — combining a leading `/` with `metadataBase` resolves against the domain root and silently drops the `/skycity` segment. `metadataBase` itself is constructed as `` new URL(`${siteConfig.url}/`) `` (trailing slash added) so any *future* relative metadata field resolves correctly instead of dropping the last path segment.

**Deploying this project**: point the Vercel project for this repo directly at the `nhungtranreal.site` domain (Project → Settings → Domains) — no rewrites, no second repo. A request to the bare domain root (`nhungtranreal.site/`, no `/skycity`) will 404 unless a redirect is added separately; that is expected with this setup, not a bug.

**GitHub / Vercel / Supabase**: this clone already has its own GitHub repo (`origin` → `github.com/lehoainamsrt-lang/nhungskycity`) and its own Supabase project, separate from the base boilerplate's — do not repoint `origin` or reuse the base project's Supabase credentials.

---
> Source: [lehoainamsrt-lang/nhungskycity](https://github.com/lehoainamsrt-lang/nhungskycity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
