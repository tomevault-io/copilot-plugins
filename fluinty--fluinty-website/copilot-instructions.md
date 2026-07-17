## fluinty-website

> > Root of the `fluinty-website` repo (separate from the AgentAI business-ops

# CLAUDE.md — fluinty-website

> Root of the `fluinty-website` repo (separate from the AgentAI business-ops
> repo). Scopes the agent to this repo only: site code, content, deploy.

## What this repo is

The production codebase for **fluinty.pl** — static HTML/CSS/JS, no
framework, no build step, no backend. Already migrated off WordPress (see
`DEPLOYMENT_GUIDE.md`). Deployed on **Netlify**, auto-deploy on every push to
`main` (live in ~30 sec). Git workflow is documented in `GIT_WORKFLOW.md` —
follow it exactly (`git pull` → edit → `git add .` → `git commit -m "..."` →
`git push`). This is a shared repo (see references to "kolega" in
`GIT_WORKFLOW.md`) — always pull before starting work.

This repo does **not** contain business context (pricing, leads, contracts,
other clients). That lives in the separate AgentAI workspace. See "Bridge to
AgentAI" below.

## Stack

- Plain HTML/CSS/JS, no React/Next/build tooling.
- Tailwind CSS via CDN (`cdn.tailwindcss.com`) + a small custom config block
  per page (colors: `primary #00C4B4`, `accent #2DD4BF`, `secondary #0B3D70`,
  `background #F8FAFC`, fonts: Inter + Space Grotesk).
- GSAP + ScrollTrigger for scroll animations, Lucide for icons
  (`lucide.createIcons()` called at the end of `<body>`).
- Cookiebot for consent management, GA4 via `gtag.js` with Consent Mode v2
  defaults set to denied.
- `index.html` — homepage
- `pages/` — standalone PL pages (case studies, product pages like
  `fluinty-debt.html` / `fluinty-fleet.html`); each is a full standalone
  HTML file copying the same `<head>` boilerplate and nav/footer markup
- `pages/blog/` — blog section (see below)
- `en/` — English mirror, same folder structure as PL (`en/pages/...`)
- `css/`, `js/`, `assets/` — shared styles, scripts, images
- Several one-off Python scripts at root (`reorder.py`, `repair_index.py`,
  `add_favicon.py`, `update_linkedin.py`, etc.) — maintenance/migration
  helpers, not a build pipeline. Don't assume any of them run automatically;
  check what a script does before running it.
- Contact form endpoint: confirm which of Formspree/EmailJS (per
  `DEPLOYMENT_GUIDE.md`) is actually wired up before reusing the pattern.

## Blog

- `pages/blog/index.html` — listing page, already created. Has a
  `<!-- POST CARD TEMPLATE -->` comment block inside `#post-grid`: copy it
  for every new post (newest first) and replace the placeholder
  "Pierwsze artykuły już wkrótce." message once the first post exists.
- Individual posts live at `pages/blog/SLUG.html`, same nesting depth as
  `index.html` (so relative paths use `../../` back to root, e.g.
  `../../css/styles.css`, `../../index.html`).
- Build each post by copying the `<head>` boilerplate + nav + footer from
  `pages/blog/index.html` (or any `pages/case-study-*.html`) to stay visually
  consistent. Use the case-study "Hero / content sections / results" layout
  patterns as a base for post body structure.

Per post, non-negotiable before pushing:
- [ ] Unique `<title>` and `<meta name="description">`
- [ ] One `<h1>`, logical h2/h3 hierarchy
- [ ] Internal links to relevant service/product pages (`fluinty-debt.html`,
      `fluinty-fleet.html`, homepage pricing section)
- [ ] `Article` JSON-LD schema in `<head>` (see `Blog` JSON-LD in
      `pages/blog/index.html` for the pattern)
- [ ] Added as a `<url>` entry in `sitemap.xml`
- [ ] Canonical `<link>` tag
- [ ] Added to the post grid in `pages/blog/index.html`
- [ ] PL page has a matching file under `en/pages/blog/` or is explicitly
      PL-only (no orphaned hreflang / dead EN switcher link)

Site-wide (check once, not per post):
- [x] `robots.txt` exists, allows crawling, points to sitemap
- [x] `sitemap.xml` exists at repo root
- [ ] `Organization` JSON-LD on the homepage (verify — add if missing)
- [ ] New pages don't regress page speed — no unoptimized images (there's no
      build step, so images must be pre-optimized before committing)

## Bridge to AgentAI (the business-ops repo)

Blog posts are **drafted in AgentAI**, not here — that's where brand voice,
offer details, pricing, and the `/seo-articles` command with full company
context live (`clients/Fluinty/brand-guide.md`, `context/fluinty.md`).

Workflow:
1. Draft + keyword research happens in AgentAI (`/seo-articles` or manual
   request).
2. Finished draft gets built into a standalone HTML file here, in
   `pages/blog/`, following the checklist above.
3. `git pull`, add the file(s), `git commit`, `git push` — Netlify deploys
   automatically.
4. Back in AgentAI, mark the source draft as published so it isn't
   regenerated later.

When both repos are connected in the same Cowork session, step 2 is done
directly — Claude reads the AgentAI draft and writes the finished HTML file
straight into this repo.

## Language

- Site copy: Polish (PL) primary, English (EN) under `/en/`
- Code, file names, commit messages: English

---
> Source: [Fluinty/fluinty-website](https://github.com/Fluinty/fluinty-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
