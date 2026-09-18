## black-pride-directory

> Black Pride Directory is a Next.js 15 static site that serves as a directory for Black pride events across the US and globally. Events are stored as Markdown files with YAML frontmatter and optionally in a Strapi CMS backend. The two sources are merged at runtime and rendered via the App Router with full SSG support.

# Black Pride Directory — Claude Code Reference

## Project Overview

Black Pride Directory is a Next.js 15 static site that serves as a directory for Black pride events across the US and globally. Events are stored as Markdown files with YAML frontmatter and optionally in a Strapi CMS backend. The two sources are merged at runtime and rendered via the App Router with full SSG support.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15.3.2 — App Router, Turbopack, SSG |
| Language | TypeScript 5 (strict mode) |
| Runtime | React 19 |
| Styling | Tailwind CSS v4 + PostCSS v4 |
| UI Components | shadcn/ui (New York style, Stone base) + Radix UI |
| Forms | React Hook Form + Zod |
| Animation | Framer Motion |
| Icons | Lucide React |
| Maps | Google Maps JS API (`@googlemaps/js-api-loader`) |
| Email | Resend API |
| CMS | Strapi v4.10+ (local dev only; no production instance yet) |
| Hosting | Vercel |

---

## Directory Structure

```
/
├── app/                        # Next.js App Router pages
│   ├── layout.tsx              # Root layout (nav, footer, analytics)
│   ├── page.tsx                # Homepage
│   ├── globals.css             # Global styles + Tailwind directives
│   ├── robots.ts               # robots.txt generation
│   ├── sitemap.ts              # sitemap.xml generation
│   ├── events/
│   │   ├── page.tsx            # Events listing — merges .md + Strapi sources
│   │   └── [slug]/page.tsx     # Event detail — SSG via generateStaticParams
│   ├── cities/
│   │   ├── page.tsx            # City listing
│   │   └── [slug]/page.tsx     # City detail
│   ├── blogs/
│   │   ├── page.tsx            # Blog listing — Strapi-only
│   │   └── [slug]/page.tsx     # Blog post detail
│   ├── post/page.tsx           # Event submission form (password-protected)
│   ├── about/page.tsx          # About page
│   └── api/post-event/route.ts # API route: forwards event submissions to Strapi
│
├── components/                 # React components
│   ├── ui/                     # shadcn/ui primitives (do not edit directly)
│   ├── events-grid-with-search.tsx  # Main events list: search, filter, view toggle
│   ├── event-card.tsx          # Individual event card
│   ├── event-form.tsx          # Event submission form (~700 lines)
│   ├── events-map.tsx          # Google Maps integration
│   ├── calendar-31.tsx         # Calendar month view
│   ├── add-to-calendar-button.tsx   # Google Calendar / iCal export
│   ├── blog-card.tsx           # Blog card
│   ├── blog-grid.tsx           # Blog grid with search + category filter
│   ├── navbar.tsx              # Top navigation
│   └── footer.tsx              # Site footer
│
├── lib/                        # Core utilities
│   ├── events.ts               # Markdown event loader (gray-matter)
│   ├── fetch.ts                # Strapi API client (fetchAll, fetchOne, postOne)
│   ├── collections.ts          # TypeScript types for all Strapi collections
│   ├── cities.ts               # Markdown city loader
│   ├── utils.ts                # Formatters, isStrapiImage guard, constants
│   ├── schemas.ts              # Zod validation schemas
│   └── actions.ts              # Next.js server actions
│
├── content/                    # Active Markdown content (READ BY APP)
│   ├── events/                 # Event .md files served by the app
│   └── cities/                 # City .md files (14 cities)
│
├── events_md/                  # STAGING — output of excel-converter.py
│                               # NOT read by the app; move files to content/events/
│
├── public/images/              # Static event and city images
│
├── excel-converter.py          # Excel → .md files (outputs to events_md/)
├── upload_to_strapi.py         # events_md/ → Strapi CMS via API
├── fetch_images.py             # Downloads og:image from event website URLs
├── patch_website_field.py      # Patches fields in existing .md files
├── blkoutlist.xlsx / .csv      # Source spreadsheet for bulk imports
│
├── next.config.ts              # Image remote patterns (11 external hosts)
├── tsconfig.json               # TypeScript config — path alias @/* → root
├── components.json             # shadcn/ui config
└── .env                        # Local secrets (never commit)
```

---

## Data Architecture

### Two Sources, One List

The events page (`app/events/page.tsx`) merges events from both sources:

```
content/events/*.md  ──── lib/events.ts (gray-matter)  ──┐
                                                           ├── merged array → EventsWithSearch
Strapi /api/events   ──── lib/fetch.ts (fetchAll)      ──┘
```

- Markdown events: slugs derived from filenames (without `.md`)
- Strapi events: `documentId` assigned as slug at merge time

Blogs are Strapi-only. Cities use both markdown (static data) and Strapi (not yet wired).

### Staging Pipeline for Bulk Imports

```
blkoutlist.xlsx
    → excel-converter.py   (generates events_md/*.md)
    → [manual review]
    → move to content/events/    ← for markdown serving
         OR
    → upload_to_strapi.py        ← for Strapi CMS
```

**Important**: `events_md/` is a staging directory. The app reads only from `content/events/`. The 86 DC Pride 2026 events in `events_md/` have not yet been moved.

---

## Event .md Frontmatter Schema

Files live in `content/events/<slug>.md`. The slug is the filename without `.md`.

```yaml
---
event_name: 'Event Name'           # string, required
location_name: 'Venue Name'        # string
street_address: '123 Main St'      # string
city: 'Washington'                 # string — city name only (no state)
city_category: 'Washington DC'     # string — used for filtering (matches city_names list)
state: 'DC'                        # string — 2-letter state code
zip_code: ''                       # string (often empty)
country: 'US'                      # string
start_date: '5/24/2026'            # string, format M/D/YYYY
end_date: ''                       # string, format M/D/YYYY (optional, for multi-day events)
start_time: '10:00 PM'             # string, format H:MM AM/PM
end_time: '4:00 AM'                # string, format H:MM AM/PM
time_zone: 'America/New_York'      # IANA timezone string
organizer: 'Organizer Name'        # string
image: 'https://...'               # string URL or empty
rsvp_required: false               # boolean
price: 0                           # number OR string (e.g. 'Free', 'Price at Door', multi-line tiers)
instagram: 'handle_without_at'     # string, no @ prefix
website: 'https://...'             # string, full URL
description: '...'                 # string, auto-generated summary
categories:                        # array of strings
  - 'LGBTQ+'
  - 'Arts & Culture'
  - 'Nightlife'
---

<!-- Optional markdown body (rarely used for events) -->
```

### Valid Categories
Defined in `lib/utils.ts`:
`LGBTQ+`, `Arts & Culture`, `Nightlife`, `Sports & Fitness`, `Food & Drink`, `Music`, `Film`, `Education`, `Community`, `Health & Wellness`, `Fashion`, `Business`

### Valid city_category Values
Defined in `lib/utils.ts` `city_names` array. Examples: `Washington DC`, `Atlanta`, `Chicago`, `Dallas`, `Houston`, `Los Angeles`, `Miami`, `New York`, `Orlando`, `Philadelphia`, `San Francisco`

---

## Strapi Collection Types

Defined in `lib/collections.ts`. All use Strapi v4.10+ dual-ID pattern (`id: number` + `documentId: string`).

| Collection | `CollectionType` enum value | Purpose |
|---|---|---|
| `Event` | `'events'` | CMS-managed events (ISO datetimes, not split date/time) |
| `BlogPost` | `'blog-posts'` | Blog content (Strapi-only) |
| `Category` | `'categories'` | Blog post categories |
| `Author` | `'authors'` | Blog post authors |
| `Organizer` | `'organizers'` | Event organizers |
| `Sponsor` | `'sponsors'` | Event sponsors |
| `City` | `'cities'` | City entries |

**Schema difference**: Strapi `Event` uses `start_datetime`/`end_datetime` (ISO 8601). Markdown `EventMetadata` uses `start_date` + `start_time` as separate strings. These are different schemas representing the same concept — keep this in mind when merging.

### Image Handling Pattern

Strapi images are objects (`StrapiImage`), while markdown images are URL strings. Always use the `isStrapiImage()` type guard from `lib/utils.ts` before rendering:

```tsx
import { isStrapiImage } from '@/lib/utils';

const src = isStrapiImage(event.image)
  ? `${process.env.NEXT_PUBLIC_STRAPI_CMS_URL}${event.image.url}`
  : event.image ?? fallbackUrl;
```

---

## Environment Variables

Set in `.env` locally; set in Vercel dashboard for deployment.

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_BASE_URL` | Full deployment URL (e.g. `https://blackpridedirectory.com`) |
| `RESEND_API_KEY` | Resend email service API key |
| `RESEND_AUDIENCE_ID` | Resend mailing list / audience ID |
| `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | Google Maps JavaScript API key |
| `STRAPI_API_TOKEN` | Read-only Strapi Bearer token (server-side fetches) |
| `STRAPI_FULL_ACCESS_API_TOKEN` | Write-access Strapi Bearer token (event submission) |
| `NEXT_PUBLIC_STRAPI_CMS_URL` | Strapi base URL — `http://127.0.0.1:1337` locally |
| `NEXT_PUBLIC_EVENT_SUBMISSION_PASSWORD` | Password gate on `/post` submission form |

---

## Development Commands

```bash
npm run dev     # Start dev server with Turbopack (http://localhost:3000)
npm run build   # Production build (also runs static generation)
npm run start   # Serve production build locally
npm run lint    # ESLint check
```

For Strapi (separate process, must be started independently):
```bash
cd <strapi-project-dir>
npm run develop   # Strapi admin at http://127.0.0.1:1337/admin
```

---

## Key Patterns & Conventions

- **App Router only** — no `pages/` directory. All routes are in `app/`.
- **Server components by default** — data fetching happens in page-level async components. Client components are marked `'use client'`.
- **Path alias** — `@/*` maps to the project root. Use `@/lib/events` not relative paths.
- **Slug derivation** — markdown event slugs come from the filename; Strapi event slugs use `documentId`.
- **Static generation** — `generateStaticParams` is used in `[slug]` routes. Build will attempt to fetch from Strapi; if Strapi is offline the build falls back to empty array (no crash, no Strapi pages).
- **shadcn/ui** — components in `components/ui/` are generated by the CLI. Do not edit them directly; regenerate via `npx shadcn@latest add <component>`.
- **Tailwind v4** — configuration is via CSS (`@theme` in `globals.css`), not `tailwind.config.js`. There is no `tailwind.config.js` file.

---

## Known Issues / In-Progress Areas

1. **`events_md/` not wired up** — 86 DC Pride 2026 events are in `events_md/` but need to be moved to `content/events/` before they appear on the site.

2. **`EventMetadata.price` type mismatch** — typed as `number` in `lib/events.ts:29` but .md files use string values (`'Free'`, `'Price at Door'`, multi-line tier strings). Should be `number | string`.

3. **Date filter commented out** — `lib/events.ts:57–65` has a past-event filter commented out. Events don't currently filter by date, so past events appear in the listing.

4. **`console.log` left in production code** — `app/events/page.tsx:8,10` logs the full events array. Remove before launch.

5. **Strapi not in production** — `fetchAll` / `fetchOne` calls in `app/events/page.tsx` and `app/blogs/` will return empty arrays on the production Vercel build since there's no deployed Strapi instance. Blog posts and Strapi-managed events won't appear in production until Strapi is hosted.

6. **No `.env.example`** — contributors must know which env vars are needed by reading the code. An `.env.example` file should be added.

7. **`city_category` vs `city`** — two city-related fields exist in the frontmatter. `city` is the plain city name; `city_category` is the filter group used by the UI. Both need to be set correctly for filtering to work.

---
> Source: [jbowen4/black-pride-directory](https://github.com/jbowen4/black-pride-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
