## cncf-darkmode

> Work only in this repository (`cncf-darkmode`) for the unified site.

# cncf-darkmode — Agent Runbook

## Canonical Scope

Work only in this repository (`cncf-darkmode`) for the unified site.

**Deprecated for this initiative:** `projects-website`, `endusers-website`, `people-website`.
Do not plan, test, or implement changes in those repos for darkmode work.

## Product Model

This is one unified Astro project (single port 4321, single `astro.config.mjs`):

- `src/pages/index.astro` — Projects section (`/cncf-darkmode/`)
- `src/pages/members/` — End Users section (`/cncf-darkmode/members/`)
- `src/pages/people/` — People section (`/cncf-darkmode/people/`)
- `src/components/` — shared components
- `src/lib/` — shared client-side logic
- `src/styles/` — shared CSS tokens and layout
- `go/` — sync binaries (sync-projects, sync-endusers, sync-people)

The `sites/`, `packages/site-kit`, `people-website`, `projects-website`, and
`endusers-website` repos are fully deprecated and deleted. Do not reference them.

## Core Rules

1. Treat the app as one site with multiple sections, not separate websites.
2. Keep navigation and header behavior consistent across sections.
3. Prefer behavior-level tests for critical UX over shallow visibility checks.
4. Keep data contracts strict (`schema.test.ts`) and avoid silent fallbacks.
5. Preserve XSS/IME safeguards already in place (`escapeHtml`, `e.isComposing`).
6. Before disabling any linked destination, add/verify a route-exists test for every SiteSwitcher target to prevent shipping navigation 404s.

## Testing — Playwright only

This repo has `playwright.config.ts`. **Never use `curl`, HTML grep, or code reading
to verify visual or rendering fixes.** That approach has silently passed broken
production states multiple times (observed violations: 2026-04-06).

```bash
npx playwright test                                    # local — playwright manages the server
npx playwright test tests/e2e/verify-prod.spec.ts     # production smoke tests
```

After every `gh workflow run deploy.yml`, wait for completion (sync-people takes 5–20 min)
then run `verify-prod.spec.ts` before declaring any fix live.

## People section — dynamic CSS rule

`src/lib/people/feed-loader.ts` and `maintainer-loader.ts` use `insertAdjacentHTML`
to inject timeline cards at runtime. Astro scoped `<style>` attaches `data-astro-cid-*`
only to SSR-rendered elements — dynamically inserted HTML gets no CID and CSS never
applies. **`PersonCard.astro` and `MaintainerCard.astro` must use `<style is:global>`.**
Do not revert to scoped `<style>`.

## Finalized design — do not modify without explicit instruction

- **Fonts** — `src/styles/variables-base.css` defines `body { font-family: ... }`. Do not touch.
- **Hero sections** — layout and per-tab hero visibility are finalized.
- **SiteSwitcher pills** — do not remove or rearrange. Each pill is 1/3 of the site's navigation.
- **`variables.css` / `layout.css`** — shared across all three sections; changes affect the whole site.

## Primary Commands

```bash
just install
just sync
just build
just test
just test-cross-site
just dev-all
```

## Validation Focus

Critical cross-section behavior is enforced by:

- `tests/cross-site/header-geometry.spec.ts`
- `tests/cross-site/search-contract.spec.ts`
- `tests/cross-site/shared-behaviors.spec.ts`

Keep these aligned with real user workflows and unified-site navigation contracts.

---
> Source: [castrojo/cncf-darkmode](https://github.com/castrojo/cncf-darkmode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
