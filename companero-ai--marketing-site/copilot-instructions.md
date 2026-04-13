## marketing-site

> *Last updated: January 2026*

# CLAUDE.md — Compañeros Marketing Site

*Last updated: January 2026*

---

## What We're Building

A marketing website for **Compañeros** (companeros.ai) — a platform that helps entrepreneurs, small business owners, and corporate leaders build products, apps, and internal tools using AI-powered guidance and proven frameworks.

This repo is the **marketing site only**. The full platform/app is a separate project.

---

## Why We're Building It

The marketing site is the front door to Compañeros. It needs to:

1. **Explain what Compañeros is** — Quickly and clearly
2. **Route visitors to the right path** — We serve three distinct audiences
3. **Build trust** — Through testimonials, credibility, and founder story
4. **Convert visitors** — Book sales, consulting bookings, email capture, platform waitlist
5. **Support both English and Spanish speakers** — Bilingual audience

---

## Who It's For

### Three Primary Audiences

| Audience | What They Want | Entry Point |
|----------|----------------|-------------|
| **Entrepreneurs & Startups** | Turn an idea into a product | Book, bundles, 2-day intensive |
| **Small Business Owners** | Build tools for their business without developers | Book, bundles, Jumpstart |
| **Corporate Leaders** | Empower their teams to innovate without IT bottlenecks | Workshops, retainers |

Each audience has a dedicated page with tailored messaging.

---

## Site Structure

```
companeros.ai/
├── / (Homepage)
│   └── "How can we help?" selector → routes to audience pages
├── /entrepreneurs
├── /small-business
├── /corporate
├── /book
├── /consulting
├── /about
├── /privacy
└── /terms
```

---

## Tech Preferences

| Decision | Preference | Notes |
|----------|------------|-------|
| **Framework** | Your choice | Next.js, Astro, or similar recommended |
| **Styling** | Your choice | Tailwind CSS recommended |
| **Hosting** | Your choice | Vercel or Netlify recommended |
| **Auth** | Not needed for marketing site | Platform (separate repo) will handle auth |
| **Payments** | Stripe | For bundles (may integrate later) |
| **Booking** | Calendly embed | For consulting scheduling |
| **Email capture** | TBD | Mailchimp, ConvertKit, or custom |
| **Analytics** | Plausible or GA | Basic traffic + conversion tracking |

Use your best judgment on technical decisions. Optimize for:
- Fast development
- Easy maintenance
- Performance (<3s load)
- SEO

---

## Key Files to Reference

| File | Contents |
|------|----------|
| `brand-guide.md` | Colors, typography, voice, logo usage |
| `ui-ux-requirements.md` | Components, layouts, spacing, interactions |
| `companeros-website-copy.md` | All page copy, ready to implement |
| `companeros-pricing-final.md` | Pricing for all products/services |
| `companeros-website-specs.md` | Wireframes, scenarios, detailed specs |

---

## Brand Summary

| Element | Value |
|---------|-------|
| **Primary Color** | Navy #1e3a5f |
| **Accent Color** | Gold #c9a227 |
| **Headline Font** | Montserrat (Bold, SemiBold) |
| **Body Font** | Inter |
| **Voice** | Warm, professional, confident, humble |
| **Languages** | English (primary), Spanish (supported) |

---

## Critical UX Requirements

### Homepage

The homepage has a **"How can we help?"** selector with three options:
1. "I'm building something new" → /entrepreneurs
2. "I need better tools for my business" → /small-business
3. "I want to drive innovation in my organization" → /corporate

This is the primary navigation pattern. Make it prominent.

### Audience Pages

Each audience page follows the same structure:
1. Hero with tailored headline
2. Problem/solution narrative
3. Demo/walkthrough area (placeholder for now)
4. Testimonials
5. Pricing/offer cards
6. FAQ
7. CTA

### Corporate Page — Key Message

The corporate page headline is:

> **"Your best people already know what to build."**

This is the centerpiece message. Everything on that page supports this idea.

### Bilingual Support

- Default: English
- "Hablamos Español" callout should appear on homepage
- Full Spanish translation is future scope, but design should accommodate

---

## Current Integrations Needed

| Integration | Purpose | Implementation |
|-------------|---------|----------------|
| **Calendly** | Consulting booking | Embed on /consulting |
| **Amazon** | Book purchase | External link |
| **Email capture** | Newsletter/waitlist | Form on homepage + footer |

### Future Integrations (Not Now)

- Stripe checkout (bundles)
- Platform login/signup
- Live chat

---

## Content Status

| Content | Status | Notes |
|---------|--------|-------|
| All page copy | ✅ Ready | See `companeros-website-copy.md` |
| Pricing | ✅ Ready | See `companeros-pricing-final.md` |
| Testimonials | 🔄 In progress | Placeholders for now |
| Book cover image | ✅ Ready | Canva design completed |
| Founder headshot | ❌ Needed | |
| Demo video | ❌ Needed | Placeholder for now |
| Logo files | ❌ Needed | Generate based on brand guide |

---

## What Success Looks Like

1. Visitor lands on homepage
2. Within 10 seconds, they understand what Compañeros is
3. They identify which audience they belong to
4. They navigate to their audience page
5. They see relevant messaging, proof, and offers
6. They take action: buy book, book consulting, or join waitlist

---

## Development Priorities

### Phase 1: Core Pages
1. Homepage with selector
2. /entrepreneurs
3. /small-business
4. /corporate
5. /consulting (with Calendly)
6. /book
7. /about

### Phase 2: Polish
- Testimonial integration
- Email capture flow
- Analytics
- SEO optimization

### Phase 3: Future
- Spanish translation
- Stripe checkout
- Platform integration

---

## Questions?

If anything is unclear, refer to:
1. `brand-guide.md` — Visual identity
2. `ui-ux-requirements.md` — Components and interactions
3. `companeros-website-copy.md` — Exact copy to use
4. `companeros-website-specs.md` — Wireframes and detailed specs

Or ask — context is in this file and the supporting documents.

---

## Contact

**Founder:** James Smith
**Email:** james@companeros.ai
**Domain:** companeros.ai

---

*Build something great. That's what we're here for.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Companero-AI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Companero-AI)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
