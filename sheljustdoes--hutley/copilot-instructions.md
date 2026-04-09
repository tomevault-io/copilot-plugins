## hutley

> This is a full website rebuild for **Hutley Van Systems**, a third-generation family-owned moving company based in Altamonte Springs, Florida, operating since 1929. The goal is to replace an outdated WordPress site (hutleyvansystems.com) hosted via Melbourne IT with a fast, modern, SEO-optimised static site.

# Hutley Van Systems — Website Rebuild Project

## Project Context

This is a full website rebuild for **Hutley Van Systems**, a third-generation family-owned moving company based in Altamonte Springs, Florida, operating since 1929. The goal is to replace an outdated WordPress site (hutleyvansystems.com) hosted via Melbourne IT with a fast, modern, SEO-optimised static site.

## Key Goals

1. **Strong Google ranking / Local SEO** — outrank competitors in Central Florida
2. **Beautiful, modern design** — not template-looking; nothing like typical WordPress sites
3. **Easy content updates** — markdown-based content, no CMS needed
4. **Zero hosting cost** — deployed to GitHub Pages
5. **Full developer control** — maintained by a data scientist comfortable with Git/code

## Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Framework | Astro | Zero-JS by default, best-in-class Core Web Vitals, built for content sites |
| Styling | Tailwind CSS v4 | Full design control, no fighting themes |
| Hosting | GitHub Pages | Free, CDN-backed, CI/CD via GitHub Actions |
| Forms | Formspree (free tier) | Quote form submissions emailed, no backend needed |
| Images | Astro Image component | Auto WebP, lazy loading, correct sizing |
| SEO | @astrojs/sitemap + custom SEO component | Auto sitemap, per-page meta, JSON-LD schema |

## Business Information

- **Company Name:** Hutley Van Systems, Inc.
- **Founded:** 1929 (Sam Hutley Sr., Quincy FL)
- **Current Operator:** Derek Hutley (grandson) and Ta'mara Hutley (great-granddaughter)
- **HQ:** Altamonte Springs, FL 32701
- **Phone:** (407) 960-0875
- **Toll Free:** (844) 331-9238
- **Fax:** (321) 972-5613
- **Email:** info@hutleyvansystems.com
- **Hours:** Mon–Sat 7:00am–6:00pm, Sun Closed
- **DOT:** 285877 | **MC:** 192877 | **IM:** 1727
- **BBB:** Accredited, Central Florida

### Leadership Team
- Derek Hutley — President
- Shelby Givens — VP of Operations
- Wanda Hutley-Geathers — Chief Financial Officer
- Lorraine Hamilton — Move Coordinator
- Carlos Hutley — Office Manager, Quincy
- Shel B. Patton — Digital Strategy & Technology

### Office Locations
- **Florida (HQ):** Altamonte Springs, FL
- **Florida:** Windermere, FL
- **Florida:** Quincy, FL
- **Mississippi:** Natchez, MS
- **New York:** Schenectady, NY

## Services

1. **Local Moves** — Central Florida residential moves
2. **Long-Distance Moves** — Door-to-door, agents nationwide
3. **Corporate Relocations** — Business moves without productivity loss
4. **Storage** — Government-inspected warehouses nationwide
5. **Logistics** — Domestic truckload, LTL, international cargo
6. **Auto Transport** — Cars, trucks, vans, boats, jet skis, motorcycles

## Service Location Pages (10 pages)

Each location page should be SEO-targeted to that city. Pages needed:
- Clermont, FL
- Deland & Deltona, FL
- Daytona Beach, FL
- Lakeland, FL
- Lake Mary, FL
- Ocala, FL
- Orlando, FL
- Sanford, FL
- Kissimmee, FL
- The Villages, FL

## Site Pages

| Page | Path | Priority |
|---|---|---|
| Home | / | P0 |
| Our Services | /our-services | P0 |
| Local Moves | /services/local-moves | P0 |
| Long Distance | /services/long-distance | P0 |
| Corporate Relocation | /services/corporate-relocation | P1 |
| Storage | /services/storage | P1 |
| Logistics | /services/logistics | P1 |
| Auto Transport | /services/auto-transport | P1 |
| Locations (index) | /locations | P1 |
| [City] Location pages (x10) | /locations/[city] | P1 |
| About | /about | P1 |
| Gallery | /gallery | P2 |
| Blog | /blog | P2 |
| Blog Post | /blog/[slug] | P2 |
| Testimonials | /testimonials | P2 |
| Get a Quote | /get-a-quote | P0 |

## SEO Strategy

### Primary Keywords to Target
- "moving companies Orlando FL"
- "movers Altamonte Springs FL"
- "local movers Central Florida"
- "long distance movers Florida"
- "corporate relocation Orlando"
- "moving company [city] FL" (x10 location variants)
- "auto transport Florida"
- "storage units moving Florida"

### Technical SEO Checklist
- [ ] Sitemap auto-generated via @astrojs/sitemap
- [ ] Google Search Console — submit sitemap after launch
- [ ] JSON-LD schema: LocalBusiness + MovingCompany type
- [ ] Open Graph tags on every page
- [ ] Canonical URLs on every page
- [ ] Alt text on every image
- [ ] robots.txt in /public
- [ ] 301 redirects from old WordPress URLs (via _redirects or GitHub Pages config)

### Google Business Profile
- Claim/update at business.google.com
- Add all services, photos, hours
- Set up post-job review request email template

## Design Direction

- **NOT** a typical WordPress/template look
- Modern, clean, trustworthy — conveys a family legacy business
- Strong use of whitespace
- Hero section with a bold value proposition
- Warm but professional colour palette (avoid generic blue/white mover clichés)
- Mobile-first — most local search is mobile
- Fast animations (Framer Motion or CSS) but never at expense of performance

## Project Status

- [ ] Astro project initialised (`npm create astro@latest`)
- [ ] Tailwind CSS configured
- [ ] Base layout + SEO component built
- [ ] Home page built
- [ ] Service pages built
- [ ] Location pages templated
- [ ] Quote form (Formspree) integrated
- [ ] Blog structure set up
- [ ] GitHub repo created
- [ ] GitHub Actions deploy workflow added
- [ ] Custom domain configured
- [ ] Google Search Console set up
- [ ] Sitemap submitted

## Key Files to Build First

1. `src/layouts/BaseLayout.astro` — HTML shell, SEO component, header/footer
2. `src/components/SEO.astro` — meta tags, OG, JSON-LD
3. `src/components/Header.astro` — nav with mobile menu
4. `src/components/Footer.astro` — contact info, links, designations
5. `src/pages/index.astro` — homepage
6. `src/pages/get-a-quote.astro` — Formspree quote form
7. `src/pages/locations/[city].astro` — dynamic location page template
8. `src/data/locations.ts` — location data (city, slug, keywords, description)

## Commands

```bash
# Install (run once)
npm create astro@latest hutley-vansystems -- --template minimal
cd hutley-vansystems
npx astro add tailwind
npx astro add sitemap

# Dev
npm run dev

# Build
npm run build

# Preview build
npm run preview
```

## GitHub Pages Deployment

Add this file as `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist
      - uses: actions/deploy-pages@v4
```

## Notes & Decisions Log

- **Melbourne IT** — domain is registered here. Keep domain there or transfer to Cloudflare (free DNS, faster). Just update A/CNAME records to point to GitHub Pages.
- **Formspree** free tier = 50 submissions/month. If quote volume exceeds this, upgrade ($10/mo) or switch to a Cloudflare Worker + Resend for full control.
- **Images** — existing images are at hutleyvansystems.com/wp-content/uploads/ — download and optimise before rebuilding. Use Astro's `<Image />` component for all of them.
- **Blog** — existing blog posts should be migrated as markdown files to `src/content/blog/`
- **Redirects** — old WordPress URLs need 301 redirects to preserve any existing SEO value. Map old slugs to new slugs in a redirects file.
- **Original site developer:** Branmarc Design, Inc. (branmarcdesign.com) — existing assets/branding may be available from them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sheljustdoes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sheljustdoes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
