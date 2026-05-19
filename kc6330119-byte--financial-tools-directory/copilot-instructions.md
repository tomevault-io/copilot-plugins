## financial-tools-directory

> `smart-investor-financial-tools.com` is one of Kevin Collins's directory sites. It was built first, then shelved due to a challenging niche. It is now being used as a UI playground to experiment with modern frontend design patterns, with the intent of applying successful patterns back to the active sites (holisticvetdirectory.com, splashpadlocator.com, seniorhomecarefinder.com, doggroomerlocator.com).

# Smart Investor Financial Tools — Redesign Session

## Project context

`smart-investor-financial-tools.com` is one of Kevin Collins's directory sites. It was built first, then shelved due to a challenging niche. It is now being used as a UI playground to experiment with modern frontend design patterns, with the intent of applying successful patterns back to the active sites (holisticvetdirectory.com, splashpadlocator.com, seniorhomecarefinder.com, doggroomerlocator.com).

## Session scope (strict)

- **In scope:** UI redesign — HTML, CSS, minimal progressive JS, template structure
- **In scope:** The homepage, as the first milestone
- **Out of scope (this session):** Airtable schema changes, build pipeline changes, new data sources, any modifications to the other four directory sites
- **Out of scope entirely:** Account creation, deploys to production, AdSense account changes

## Stack

Python static site generation → Airtable (read-only CMS) → Netlify hosting → Google AdSense monetization. See the `smart-investor-redesign` skill for full stack conventions.

## Runbooks (cross-site methodology)

When the user reports a search-engine indexing alert, performance question, or asks "is something wrong with our SEO" — read the SEO diagnostic runbook before starting:

- `~/notes/claude-runbooks/SEO_DIAGNOSTIC_RUNBOOK.md` — playbooks for GSC indexing alerts (noindex / alternate canonical / 404), single-URL 5xx errors, open-ended SEO health checks, Bing-specific diagnosis, meta description audits, and IndexNow integration. Built from the 2026-05-11 tuning session and intended to transfer across all 5 directory sites.

The runbook is centralized at `~/notes/claude-runbooks/` so the same playbook serves all Kevin's directory sites without drift.

## Skills to use on this project

At session start, load skills in this order when relevant:

1. **`smart-investor-redesign`** (personal skill) — stack constraints, AdSense rules, visual direction. Read this first; it defines the guardrails.
2. **`frontend-design`** — typography, spacing, component taste, distinctive aesthetic.
3. **`web-design-guidelines`** (if installed) — modern layout primitives and patterns.

If the `smart-investor-redesign` skill contradicts a general design skill, the personal skill wins — it encodes project-specific constraints.

## Design direction

Finance-native: Bloomberg / Kraken Pro / Robinhood / Koyfin as reference points. Data-dense, confident typography, semantic color (green/red/amber), dark-mode-friendly. Full details in the skill's `references/finance-native-direction.md`.

## First milestone

Redesign the homepage only. Before writing new CSS:
1. Inventory the current codebase
2. Identify all AdSense slots and preserve them
3. Produce a one-page plan describing the proposed homepage structure, typography system, and color palette
4. Wait for Kevin's approval before implementing

## Decision log

All non-obvious design decisions get a one-line entry in `REDESIGN_NOTES.md` in the project root. Create this file on day one and keep it current.

## Commit conventions

- Small, scoped commits
- Commit messages describe the design intent, not just the file change
- Redesign work happens on a branch (`redesign/homepage-finance-native` or similar), never directly on main
- Do not commit generated build output (verify the existing `.gitignore` covers `build/`, `dist/`, or `public/` as appropriate)

## Deploy

Do not deploy during this session unless Kevin explicitly asks. Netlify auto-deploys from main by default; that's why redesign work stays on a branch.

## Things to ask before doing

- Adding any new dependency (npm, Python package, or CDN script)
- Changing any URL, slug, or canonical tag
- Moving or removing an AdSense slot
- Any change that touches the build pipeline
- Any work on the other four directory sites

## Kevin's working style

- Prefers explicit plans before implementation for anything non-trivial
- Values a decision log
- Works across Mac mini, MacBook Pro, and Windows — use cross-platform paths and shell commands; when platform matters, ask
- ITIL-minded — expects clear rollback paths for changes

---
> Source: [kc6330119-byte/financial-tools-directory](https://github.com/kc6330119-byte/financial-tools-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
