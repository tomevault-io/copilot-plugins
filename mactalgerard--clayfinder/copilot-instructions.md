## clayfinder

> ClayFinder (clayfinder.com) is a niche directory website for pottery and ceramics studios

# ClayFinder — Claude Code Context

## What This Project Is

ClayFinder (clayfinder.com) is a niche directory website for pottery and ceramics studios
across the US, Canada, and Australia. It surfaces enriched studio listings from
a Supabase database populated by the `pottery-directory` data pipeline.

The goal: rank for "pottery classes near me" and related keywords, generate leads for studios,
and monetize via display ads and featured listings.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Framework | Next.js 14+ (App Router) |
| Styling | Tailwind CSS + shadcn/ui (Nova/Radix preset) |
| Database | Supabase (shared with pottery-directory pipeline) |
| Hosting | Vercel |
| Email (leads) | Resend (wired up — sends to gerardmactal@germacdirectories.com) |
| Maps | Google Maps iframe embed |

---

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
RESEND_API_KEY=your-resend-api-key
```

Set these in `.env.local` for local dev and in Vercel dashboard for production.

---

## Repo Structure

```
app/
  page.tsx                                    ← Homepage (hero + search + 3-country selector + popular cities)
  layout.tsx                                  ← Root layout with Navbar
  sitemap.ts                                  ← Auto-generated sitemap.xml from Supabase (US + AU + CA)
  robots.ts                                   ← robots.txt (allows all, blocks /api/)
  components/
    Navbar.tsx                                ← Sticky top nav with ClayFinder logo
    SearchBar.tsx                             ← Client-side city autocomplete search
  pottery-classes/
    us/
      page.tsx                                ← US landing page — states grid
    [state]/
      page.tsx                                ← US state page — city grid
      [city]/
        page.tsx                              ← US city page — listing cards
        [slug]/
          page.tsx                            ← US listing page (main page type)
          LeadForm.tsx                        ← Client-side lead capture form (shared across all countries)
    au/
      page.tsx                                ← Australia landing page — states/territories grid
      [state]/
        page.tsx                              ← AU state page — city grid
        [city]/
          page.tsx                            ← AU city page — listing cards
          [slug]/
            page.tsx                          ← AU listing page
    ca/
      page.tsx                                ← Canada landing page — provinces grid
      [state]/
        page.tsx                              ← CA province page — city grid
        [city]/
          page.tsx                            ← CA city page — listing cards
          [slug]/
            page.tsx                          ← CA listing page
  api/
    contact/route.ts                          ← Lead form handler — stores to Supabase + emails via Resend
lib/
  supabase.ts                                 ← Supabase client (anon key)
  types.ts                                    ← Listing TypeScript interface
  slugify.ts                                  ← slugify() and deslugify() utilities
```

---

## URL Structure

```
/                                             ← Homepage
/pottery-classes/us/                          ← US landing page (states grid)
/pottery-classes/[state]/                     ← US state page (e.g. /pottery-classes/california)
/pottery-classes/[state]/[city]/              ← US city page (e.g. /pottery-classes/california/los-angeles)
/pottery-classes/[state]/[city]/[slug]/       ← US listing page
/pottery-classes/au/                          ← Australia landing page (states grid)
/pottery-classes/au/[state]/                  ← AU state page (e.g. /pottery-classes/au/new-south-wales)
/pottery-classes/au/[state]/[city]/           ← AU city page
/pottery-classes/au/[state]/[city]/[slug]/    ← AU listing page
/pottery-classes/ca/                          ← Canada landing page (provinces grid)
/pottery-classes/ca/[state]/                  ← CA province page (e.g. /pottery-classes/ca/ontario)
/pottery-classes/ca/[state]/[city]/           ← CA city page
/pottery-classes/ca/[state]/[city]/[slug]/    ← CA listing page
```

Slugs are generated with `slugify()` — lowercase, hyphens, alphanumeric only.
Lookups use Supabase `ilike` on state + city, then JS `.find()` by slug match on name.

---

## Data Source

All listing data comes from the `listings` table in Supabase, populated by the
`pottery-directory` pipeline. The table has a composite PK: `(name, postal_code, country)`.

**Current data:**
- US: 1,993 listings as of 2026-04-05, 67.5% enriched
- AU: ~359 listings imported as of 2026-04-06 — data quality issues (mixed US state names, abbreviated AU state names); pipeline needs re-run before AU is production-ready
- CA: imported as of 2026-04-07 — routing live, data quality unknown

Key fields used by the website:

| Field | Used on |
|---|---|
| `name` | All pages |
| `city`, `state`, `country` | Routing + page headers |
| `full_address`, `phone`, `email`, `website` | Listing page |
| `latitude`, `longitude` | Google Maps embed |
| `working_hours` | Listing page (parsed from Python dict string) |
| `business_status` | Open badge |
| `reviews_count` | Review count display |
| `description` | Listing page + city page cards |
| `class_types`, `skill_levels` | Badges on listing + city pages |
| `price_range` | `$` / `$$` / `$$$` display |
| `studio_type` | Badge |
| `drop_in_available`, `open_studio_access`, `kids_classes`, `private_events`, `byob_events`, `date_night`, `firing_services`, `sells_supplies`, `booking_required` | Feature pills |
| `membership_model` | Membership section |

---

## Supabase Tables

### `listings` (read-only from website)
Source: pottery-directory pipeline. RLS enabled with public SELECT policy.

### `leads` (write from website)
Stores lead form submissions from listing pages.

```sql
create table leads (
  id           bigserial primary key,
  name         text not null,
  email        text not null,
  phone        text,
  message      text not null,
  studio_name  text not null,
  submitted_at timestamptz not null
);
```

RLS enabled with public INSERT policy.

---

## Key Implementation Details

### Rendering Strategy
- All pages use `export const revalidate = 86400` (ISR, 24h)
- `generateStaticParams` on all dynamic routes — pre-generates state, city, and listing pages at build time
- `dynamicParams = true` (default) — unknown slugs rendered on demand

### Hours Parsing
OutScraper stores hours as a Python dict string with list values:
`{'Monday': ['9a.m.-5p.m.'], 'Tuesday': ['Closed']}`

Some AU listings have multi-slot days: `{'Friday': ['10AM-2PM', '6-8:30PM']}`.

The `parseHours()` function in both listing pages converts single quotes to double quotes,
then collapses array values — joining multiple slots with ` / ` (e.g. `10AM-2PM / 6-8:30PM`).
Days are sorted Monday → Sunday. **Note:** `parseHours()` is duplicated in both the US and AU
listing pages — not extracted to `lib/`.

### Slug Lookup
The listing page queries all listings for a city/state, then finds by slug:
```typescript
data.find(l => slugify(l.name) === params.slug)
```
No slug column in Supabase — computed at runtime. Works fine for city-level result sets.

### Lead Form
- Client component (`LeadForm.tsx`) — POSTs to `/api/contact`
- API route stores to Supabase `leads` table
- Email delivery via Resend — sends notification to `gerardmactal@germacdirectories.com` from `leads@clayfinder.com`

### LocalBusiness JSON-LD Schema
- Added to all listing pages via `buildJsonLd()` in each country's listing page file
- Fields: name, description, telephone, url, priceRange, address (PostalAddress), geo (GeoCoordinates)
- `addressCountry` is hardcoded per route: `'US'`, `'AU'`, or `'CA'`
- Confirmed valid via Google Rich Results Test (2 valid items: LocalBusiness + Organization)

### BreadcrumbList JSON-LD Schema
- Added to all listing pages (US, AU, CA) alongside LocalBusiness schema
- Enables breadcrumb rich results in SERPs
- US listing breadcrumb: Home → United States → State → City → Listing
- AU listing breadcrumb: Home → Australia → State → City → Listing
- CA listing breadcrumb: Home → Canada → Province → City → Listing

### FAQPage JSON-LD Schema
- Added to US, AU, and CA landing pages (`/pottery-classes/us`, `/pottery-classes/au`, `/pottery-classes/ca`)
- 4 FAQs per page, country-specific questions (pricing in local currency, relevant terminology)
- Targets "People Also Ask" rich results in SERPs

### Trust Pages + Footer
- `app/about/page.tsx`, `app/contact/page.tsx`, `app/privacy/page.tsx`, `app/terms/page.tsx`
- `app/components/Footer.tsx` — sitewide footer with links to all trust pages + country browse links
- All four pages included in `app/sitemap.ts`
- Contact email: `hello@clayfinder.com`

### Thin Content Fix
- All state/province pages (US, AU, CA): added intro paragraph using keyword cluster
- All city pages (US, AU, CA): added intro paragraph with keyword cluster
- Fixed misused `<h2>` subtitle tags on state/city pages (downgraded to `<p>`)

### Breadcrumb Hierarchy Fix
- US state, city, and listing pages now include "United States" as intermediate breadcrumb step
- Consistent with AU (`Home → Australia → State → ...`) and CA (`Home → Canada → Province → ...`) patterns

---

## SEO Foundations

### Keyword Cluster
- Primary: `pottery classes near me` (KD 4, 110k SV/mo)
- Co-primary: `ceramics classes near me` (KD 3, 110k SV/mo)
- Total cluster SV: ~260k+/month
- Full cluster details: see `context/keyword_cluster.md` in the pottery-directory repo

### On-Page SEO Applied
- One H1 per page, always keyword-containing
- `generateMetadata()` on every page type with keyword-rich titles + descriptions
- Breadcrumbs on all listing and city pages with internal links up the hierarchy
- `sitemap.xml` auto-generated from Supabase data (`app/sitemap.ts`)
- `robots.txt` allows full crawl, blocks `/api/` (`app/robots.ts`)
- LocalBusiness JSON-LD schema on all listing pages
- Google Analytics (GA4): `G-HDM6FL7HQ6` — added to `app/layout.tsx`
- Google Search Console: domain verified, sitemap submitted

### Meta Title Patterns
| Page | Pattern |
|---|---|
| Homepage | `Find Pottery and Ceramics Classes Near Me \| ClayFinder` |
| State | `Pottery Classes in [State] \| ClayFinder` |
| City | `Pottery Classes in [City], [State] \| ClayFinder` |
| Listing | `[Business Name] — Pottery Classes in [City], [State] \| ClayFinder` |

---

## Session Log

### 2026-04-05 — Launch session
- Confirmed build passing cleanly (1,993 US listings, 35 states, 259+ cities, 835+ listing pages)
- Deleted `app/api/debug/route.ts` (pre-launch cleanup)
- Deployed to Vercel (mactalgarad/clayfinder repo, Hobby plan)
- Fixed env var typo in Vercel dashboard: `NEXT_PUBLIC_SUPBASE_URL` → `NEXT_PUBLIC_SUPABASE_URL`
- Added `clayfinder.com` and `www.clayfinder.com` as custom domains in Vercel
- Updated Porkbun DNS: replaced ALIAS with A record (`@` → `216.198.79.1`), added CNAME (`www` → Vercel)
- **clayfinder.com is now live**

### 2026-04-05 — Post-launch session
- Fixed `app/layout.tsx` metadata (was still "Create Next App" placeholder)
- Added GA4 tracking (`G-HDM6FL7HQ6`) to `app/layout.tsx` — confirmed firing via realtime report
- Added Google Search Console HTML verification meta tag — domain verified, sitemap submitted
- Added LocalBusiness JSON-LD schema to listing pages — confirmed valid via Google Rich Results Test
- Added Canada/Australia country selector to homepage — USA active, CA/AU greyed out "Coming soon"
- Wired up Resend email delivery in `/api/contact/route.ts` — verified `clayfinder.com` domain in Resend, lead notifications send from `leads@clayfinder.com` to `gerardmactal@germacdirectories.com`
- Added `RESEND_API_KEY` to `.env.local` (also needs adding to Vercel env vars before deploying)

### 2026-04-06 — Australia launch session
- Added full AU routing under `/pottery-classes/au/` — 4 new pages: AU landing, AU state, AU city, AU listing
- AU listing page reuses `LeadForm` from the US route; `addressCountry: 'AU'` set in JSON-LD
- Homepage Australia card activated (live link + real studio/state counts from Supabase); Canada remains "Coming soon"
- Sitemap refactored into `buildPages()` helper — now generates URLs for both US and AU listings
- Fixed `parseHours()` bug in both listing pages: multi-slot days (e.g. `['10AM-2PM', '6-8:30PM']`) were producing invalid JSON and falling back to raw string display; now joined with ` / `
- Identified AU data quality issues: some listings have US state names or abbreviated AU state names — AU pipeline needs re-run and clean import before AU should be promoted

### 2026-04-07 — Canada launch + homepage rework session
- Added full CA routing under `/pottery-classes/ca/` — 4 new pages: CA landing, CA province, CA city, CA listing (mirrors AU pattern exactly); `addressCountry: 'CA'` in JSON-LD
- Added `/pottery-classes/us/` landing page (states grid) to give US symmetric URL structure with AU and CA
- Homepage redesigned: removed US states grid, replaced with 3-country selector (US/CA/AU all live with real counts) + Popular Cities section (8 US + 2 AU + 2 CA cities linking directly to city pages)
- Sitemap updated: includes `/pottery-classes/us/`, `/pottery-classes/ca/`, and all CA state/city/listing URLs
- Discussed homepage content — decided against history/why-pottery editorial; Popular Cities preferred as it targets search intent and adds SEO-valuable internal links
- AU and CA data quality issues fixed (pipeline re-run completed)

### 2026-04-07 — SEO foundations session
- Added trust pages: About, Contact, Privacy Policy, Terms of Service + sitewide footer with links
- Added trust pages to sitemap (`app/sitemap.ts`)
- Added intro/body copy to all state and city hub pages across US, AU, CA (thin content fix)
- Fixed misused `<h2>` subtitle tags on state/city pages
- Added FAQPage JSON-LD schema + FAQ sections to US, AU, CA landing pages (4 country-specific FAQs each)
- Added BreadcrumbList JSON-LD schema to all listing pages (US, AU, CA)
- Fixed US breadcrumb hierarchy to include "United States" intermediate step
- Created `scripts/seo_audit.py` — DataForSEO On-Page API audit script (~$0.75–$1.50/run, 500 pages)
- Ran DataForSEO audit; fixed real issues:
  - Added `alternates.canonical` to all 17 page types (was missing entirely — Next.js does not auto-generate canonicals without this)
  - Added fallback description to listing pages without a database description (built from name, city, class types, phone)
  - Fixed duplicate React key warning on city pages — changed `key={listing.name}` to `key={listing.name + postal_code}` to handle studios with identical names in the same city
- Ignored false positives from audit: `is_https`, `is_www`, `has_html_doctype`, `has_render_blocking_resources`, `frame`, `https_to_http_links`, `seo_friendly_url` variants
- `title_too_long` (199 pages) — intentionally left; Google truncates SERP display but still uses full title for ranking

---

## TODO — Next Steps

### Immediate (done)
- [x] Wire up Resend for lead email delivery
- [x] Add Google Analytics (GA4)
- [x] Google Search Console: verify domain + submit sitemap
- [x] Add LocalBusiness JSON-LD schema to listing pages
- [x] Add `RESEND_API_KEY` to Vercel environment variables + redeploy
- [x] Add trust pages (About, Contact, Privacy, ToS) + sitewide footer
- [x] Fix thin content: add intro copy to all state and city hub pages
- [x] Add FAQPage schema + FAQ sections to US, AU, CA landing pages
- [x] Add BreadcrumbList schema to all listing pages
- [x] Fix US breadcrumb hierarchy
- [x] AU and CA data quality fixed (pipeline re-run done)

### SEO & Technical (in progress / next)
- [x] Review DataForSEO audit report and fix real issues (canonical tags, thin content, duplicate keys)
- [ ] Monitor Search Console Coverage report (3–5 days after sitemap submission)
- [ ] Verify FAQPage and BreadcrumbList schema via Google Rich Results Test
- [ ] Deploy all changes to Vercel

### Growth (weeks 2–4+)
- [ ] Start backlink outreach (2–3 weeks post-launch, after checking Search Console for early rankings)
  - Strategy: link swaps with competing directories
  - Reddit mentions: r/Pottery, r/Ceramics
- [ ] Design polish pass
- [ ] User auth + business dashboard (claimed listings — natural upsell once leads flowing)
- [ ] Apply for display ads once traffic threshold is reached (Google AdSense first, Mediavine at ~50k sessions/month)

---

## Competitors to Monitor

| Competitor | Type | Threat level |
|---|---|---|
| ClassBento (classbento.com) | Paid booking marketplace | Medium — different model, strong in major metros |
| KilnFire.com | Thin directory | Low — beatable on content depth |
| Yelp | General directory | Low — not niche-focused |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mactalgerard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mactalgerard)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
