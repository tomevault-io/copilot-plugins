## first-years-foundations

> This file provides guidance to Claude Code when working on the First Year Foundations website.

# CLAUDE.md — First Year Foundations

This file provides guidance to Claude Code when working on the First Year Foundations website.

## Project Overview

**First Year Foundations** is a digital product business selling Feldenkrais-informed baby development guides (PDF format) to parents. Static HTML website hosted on GitHub Pages with custom domain `first-year-foundations.com`.

**Current Status:**
- ✅ Homepage (index.html) complete
- ✅ GitHub Pages deployment configured
- ✅ Design system in place (CSS variables, components)
- ❌ 13 pages remain to build
- ❌ 2 blog posts to write
- ❌ 3 legal pages (disclaimer, terms, privacy)
- ❌ sitemap.xml and robots.txt

## AI Team

This project is run day-to-day by a small crew of Claude Code subagents (see
`.claude/agents/`: `ceo`, `hr-recruiter`, `content-marketer`). Start with
`.claude/company/charter.md` (operating rules/approval gate) and
`.claude/company/org-chart.md` (current roster) — the crew grows over time via
`hr-recruiter`'s hiring process, gated through founder approval.

## Tech Stack

- **Hosting:** GitHub Pages (static site)
- **Framework:** Vanilla HTML/CSS/JS (no build step)
- **Repository:** https://github.com/talnoga-png/first-years-foundations (main branch → auto-deploys)
- **Website domain:** first-year**s**-foundations.com — WITH the "s" (GitHub Pages custom
  domain, per CNAME + all page canonicals). This is where the live site is served.
- **Contact email:** hello@first-year-foundations.com — NO "s" (Porkbun-forwarded mailbox,
  MailerLite-verified sender). The website domain and the email domain deliberately differ;
  do not "unify" them.
- **E-commerce:** Whop (external — guides sold via Whop store)
- **Email:** MailerLite (automation sequences for leads + buyers)

## Build Workflow

Pages are generated from `*.src.html` source files via a small Node build script. **Never edit
the generated `.html` files directly** — edits will be overwritten by the next build.

1. **Nav/footer edits** → edit `_nav.html` / `_footer.html` (shared partials, single source of truth)
2. **Content edits** → edit the relevant `<page>.src.html` file
3. **Styles** → edit `styles.css` (shared stylesheet, includes a legacy CSS-variable alias block
   for old `var(--purple-main)`-style names used throughout the page content)
4. **Behavior** → edit `site.js` (shared script: mobile nav burger, FAQ accordion, scroll-reveal
   via IntersectionObserver)
5. **Run the build:** `node build.js` — regenerates all `*.html` files from `*.src.html`,
   injects `{{NAV}}`/`{{FOOTER}}`, and sets `data-page` on `<body>` (add new pages to the
   `PAGE_IDS` map in `build.js` if needed)
6. **Test locally** by serving the directory (e.g. `npx serve .`) and checking at 390px + 1280px
7. **Git commit & push** (both `.src.html` sources AND the regenerated `.html` files) →
   GitHub Pages auto-rebuilds site
8. **Commit message format:** `feat(page-name): add [page title]` or `docs: update [page name]`

## Design System

Shared stylesheet: `styles.css`. Key CSS variables:

- Colors: `--purple-dark`, `--purple-button`, `--lavender-bg`, `--teal`, etc. (legacy aliases
  for these names are kept in `styles.css` for backward compatibility with inline styles)
- Layout: `.container` (max-width 900px), `.container--wide` (1100px)
- Typography: Lato (sans-serif stack), 17px base, 1.7 line-height
- Components: `.btn`, `.guide-card`, `.why-card`, `.faq-item`/`.faq-q`/`.faq-a`, etc.
- Scroll-reveal: add `.reveal` (single elements) or `.reveal-stagger` (grid/list containers)
  to animate sections in on scroll (handled by `site.js`)

## Content Structure

All page content is provided in markdown format ready to convert to HTML. The brief document contains:

- **Product pages:** 0–3, 3–6, 6–9, 9–12 months guides + bundle
- **Information pages:** About, FAQ, free guide opt-in
- **Blog:** Index page + 2 priority posts (tummy time, rolling)
- **Legal:** Health disclaimer, terms, privacy policy
- **SEO:** Every page needs unique title, meta description, canonical, Open Graph, and JSON-LD schema

## Pages to Build (in priority order)

| Page | Path | Status | Notes |
|------|------|--------|-------|
| About | /about.html | TO BUILD | Trust page — build early |
| 0–3 Months Guide | /0-3-months.html | TO BUILD | Product page → Whop URL |
| 3–6 Months Guide | /3-6-months.html | TO BUILD | Product page → Whop URL |
| 6–9 Months Guide | /6-9-months.html | TO BUILD | Product page → Whop URL |
| 9–12 Months Guide | /9-12-months.html | TO BUILD | Product page → Whop URL |
| Bundle | /bundle.html | TO BUILD | Product page → Whop URL |
| Free Guide Opt-In | /free.html | TO BUILD | Embed MailerLite form |
| Blog Index | /blog.html | TO BUILD | List blog posts |
| Blog Post 1 | /blog/tummy-time-newborns.html | TO BUILD | Link to /0-3-months.html |
| Blog Post 2 | /blog/support-baby-rolling.html | TO BUILD | Link to /3-6-months.html |
| Health Disclaimer | /disclaimer.html | TO BUILD | Legal page |
| Terms of Use | /terms.html | TO BUILD | Legal page |
| Privacy Policy | /privacy.html | TO BUILD | Legal page |
| sitemap.xml | /sitemap.xml | TO BUILD | All pages listed |
| robots.txt | /robots.txt | TO BUILD | Allow all, point to sitemap |

## Important Details

**SEO Requirements (every page):**
- Unique `<title>` (140–160 chars, format: "[Topic] | First Year Foundations")
- Unique meta description (140–160 chars, outcome-focused)
- Self-referencing canonical tag
- Open Graph: og:title, og:description, og:image, og:url, og:type
- JSON-LD schema (Product, Person, BlogPosting, FAQPage, etc.)
- Credential byline: "Written by a Feldenkrais-certified practitioner"
- Educational disclaimer on content pages

**Product Pages:**
- Include Whop product URL (placeholder: `[INSERT WHOP X PRODUCT URL]`)
- Show pricing
- Include FAQ block with product-specific Q&A
- Add educational disclaimer

**Blog Posts:**
- Quick answer block at top (2–3 sentences)
- Outbound links to NHS, AAP, WHO, etc. where relevant
- Link back to relevant guide page
- Author: "First Year Foundations Team, Feldenkrais-certified practitioner"
- Published date

**Tone & Voice:**
- Calm, observant, reassuring, knowledgeable, non-alarming
- Never use: diagnose, treat, cure, therapy, fix, delay, "will" + outcome promise
- Use: "Many babies explore this in different ways", "Development often unfolds with variation", "Your baby is not behind"

## Whop Product URLs

You'll need to fill in these placeholders from your Whop dashboard:

- `[INSERT WHOP 0-3 PRODUCT URL]` → 0–3 months guide
- `[INSERT WHOP 3-6 PRODUCT URL]` → 3–6 months guide
- `[INSERT WHOP 6-9 PRODUCT URL]` → 6–9 months guide
- `[INSERT WHOP 9-12 PRODUCT URL]` → 9–12 months guide
- `[INSERT WHOP BUNDLE URL]` → First Year Bundle

## MailerLite Setup

Free guide opt-in page needs to embed the MailerLite form. URL:
`first-years-foundations-ndpk6k.subscribepage.io`

On /free.html, replace `<!-- EMBED MAILERLITE OPT-IN FORM HERE -->` with the embed code from MailerLite.

## After Website Build

Before launch:
- [ ] Delete TESTORDER promo code in Whop
- [ ] Add real PDF guides to Whop (currently placeholders)
- [ ] Lawyer review of legal pages
- [ ] Create product banners in Canva for Whop
- [ ] Write blog posts and create Pinterest pins
- [ ] Add first-year-foundations.com to Google Search Console
- [ ] Submit sitemap.xml

## Commands

```bash
# Edit *.src.html / _nav.html / _footer.html / styles.css / site.js
# Then rebuild the generated .html pages:
node build.js

# To preview locally:
npx serve .

# Git workflow (commit both .src.html sources and regenerated .html):
git add [files]
git commit -m "feat(page-name): add [page title]"
git push origin main
```

## How to Continue in Future Sessions

Start your message with:

"I'm continuing work on First Year Foundations. I'm [building page X / writing blog post Y / etc.]. The Master Build Brief has all content + context."

Claude will search the brief and have full context immediately.

---
> Source: [talnoga-png/first-years-foundations](https://github.com/talnoga-png/first-years-foundations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
