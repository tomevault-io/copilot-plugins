## nexalforge-landingpage

> - **Framework:** Next.js 15.4.10 (App Router) + React 19.1.0

# CLAUDE.md - Nexal Forge Project Context

## Project Overview

### Tech Stack
- **Framework:** Next.js 15.4.10 (App Router) + React 19.1.0
- **Styling:** Tailwind CSS v4 with PostCSS
- **Language:** TypeScript 5 (strict mode)
- **Database:** PostgreSQL via Prisma 6.16.2
- **Animations:** Framer Motion 12.23.12
- **Forms:** React Hook Form + Zod validation
- **Analytics:** GA4, GTM (Consent Mode v2), Facebook Pixel, Vercel Analytics

### Key Directories
```
/src/app/          # Next.js App Router pages
/src/components/   # React components
/src/lib/          # Utilities and helpers
/src/data/         # Static data files
/prisma/           # Database schema
```

---

## The "Nexi" Definition

**CRITICAL:** Nexi is positioned as an **Agentic OS** (handling bookings, routing, tasks), NOT just a chatbot.

### Current Positioning Problem
The current landing page **fails to communicate this**:
- The term "Agentic OS" appears **nowhere** on the site
- Nexi is presented as "one of many agents" rather than the core platform
- Visitors may confuse Nexi with a simple chatbot

### What Nexi Actually Is
- A complete **operating system for AI agents**
- Handles: bookings, task routing, email automation, WhatsApp support
- Target: LATAM businesses (clinics, law firms, e-commerce, real estate)

---

## Technical SEO Gap Analysis

| Element | Status | Issue |
|---------|--------|-------|
| Dynamic Metadata | ✓ Partial | Root + some pages have metadata; not all solution pages |
| Schema.org | ⚠ Limited | Only Organization, Service, FAQPage. **Missing:** SoftwareApplication, Product, Article |
| Sitemap | ✓ Implemented | Dynamic but force-dynamic (no caching) |
| Robots.txt | ✓ Implemented | Basic allow-all |
| Alt Tags | ⚠ Partial | Hero images have alt, blog covers use raw `<img>` without proper alt |
| Canonical URLs | ✓ Implemented | Present on main pages |
| Open Graph | ✓ Good | Root + page-level OG tags |
| hreflang | ✗ Missing | No language/region targeting despite LATAM focus |

### Missing Schema Types to Implement
1. **SoftwareApplication** - For Nexi product pages
2. **Article** - For blog posts
3. **Product** - For pricing/feature pages

---

## Performance Bottlenecks (Core Web Vitals)

### LCP Issues
- Hero images are 1.6-1.9MB each (not WebP optimized)
- Video autoplay may compete with LCP
- **Fix:** Convert to WebP, add `priority` prop to hero images

### CLS Risks
- Blog covers use raw `<img>` instead of Next/Image (no dimensions)
- External images in SafeImage component set `unoptimized={true}`
- **Fix:** Replace `<img>` with `<Image>` component, specify dimensions

### JavaScript Bundle
- Blog page is 672 lines of client-side code with heavy Framer Motion
- 6 `"use client"` components
- **Fix:** Consider code splitting, lazy loading animations

---

## Current Data Reality (GSC Performance)

### Overall Stats (Past 4+ months)
- Total Impressions: 2,036
- Total Clicks: 78
- CTR: 3.8%
- Average Position: 7.84

### Critical Issues
1. **Branded keywords only:** "nexal" variants = 89% of impressions
2. **Zero ranking for target keywords:**
   - "agentes ia para empresas" (1K-10K vol)
   - "automatización secretaria" (500-1K vol)
3. **Colombia underperformance:** 1,182 impressions, 0% CTR
4. **Tablet black hole:** 1,155 impressions, 0% CTR

### Opportunity Gap
- Target keywords: 4.5K-12.5K monthly searches
- Currently ranking for: 0 of 9 primary keywords
- Potential traffic increase: 10-100x current volume

---

## Key Files Reference

### SEO Configuration
- `/src/app/layout.tsx` - Root metadata
- `/src/app/sitemap.ts` - Dynamic sitemap
- `/src/app/robots.ts` - Robots configuration
- `/src/components/JsonLd.tsx` - Schema.org markup

### Components to Modify for SEO/CRO
- `/src/components/ContactForm.tsx` - Add phone field
- `/src/components/HeroText.tsx` - Reposition "Agentic OS" messaging
- `/src/app/blog/[slug]/page.tsx` - Fix image optimization

### Data Files
- `/src/data/services.ts` - Service definitions
- `/src/data/pSEO.json` - (To create) pSEO data schema

---

## Development Commands

```bash
# Development
npm run dev

# Build
npm run build

# Prisma
npx prisma generate
npx prisma db push
npx prisma studio
```

---

## SEO Strategy Reference

See `/GROWTH_PLAN.md` for:
- Programmatic SEO (pSEO) architecture
- Technical implementation guides
- CRO recommendations
- Prioritized roadmap

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SebasChicaiza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
