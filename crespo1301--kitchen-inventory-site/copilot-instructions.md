## kitchen-inventory-site

> Guidance for Codex and other coding agents working in this repository.

# AGENTS.md

Guidance for Codex and other coding agents working in this repository.

## Project Role

Marketing site for **Kitchen Inventory**, a restaurant stock & ordering app
built in the sibling `inventory-app` repo. This repo is the public-facing
site at the bare product domain; the app's own web export lives at the
`app.` subdomain. See that repo's `docs/launch-roadmap.md` Phase 5 for why
this split exists.

## Stack

Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4. Node 20+.

## Content Source

- `src/data/site.ts` holds all site copy and links in one place — edit
  copy there, not scattered across components.
- Copy should stay aligned with `inventory-app/docs/app-store-listing.md`
  (the App Store / Play Store listing draft) — same product description,
  same tone, not a rewrite.
- `TODO` markers in `site.ts` (domain, app URL, store links, support email)
  are real placeholders — do not remove them without a confirmed real value.
  Never invent a domain or store URL.

## Boundaries

- No backend, no auth, no forms that collect personal data without a
  documented reason — this site does not need a privacy policy of its own
  scope beyond what's already covered by `inventory-app/docs/privacy-policy.md`
  unless the site itself starts collecting data (e.g. a contact form).
- Keep this site functionally real once live — both Apple and Google check
  it during Organization verification (`inventory-app/docs/launch-roadmap.md`
  Phase 0). Don't ship placeholder/lorem content to production.
- Do not register or purchase the domain without Carlos's explicit approval.

## Visual QA

Use the workspace runner at `/home/cresp3/scripts/visual-check.sh` after any
layout, responsive, spacing, or visual-polish change. Start the local dev
server, capture mobile and desktop screenshots into `.visual-checks/`, and
inspect the rendered pixels before calling the work done.

---
> Source: [Crespo1301/kitchen-inventory-site](https://github.com/Crespo1301/kitchen-inventory-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
