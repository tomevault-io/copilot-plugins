## trait

> Inherits all behavior from `/AGENTS.md` at the monorepo root. Local rules extend or override the root file for this repository.

# Trait - Agent Configuration

## Root Configuration

Inherits all behavior from `/AGENTS.md` at the monorepo root. Local rules extend or override the root file for this repository.

## Project Context

This repository is the Astro-powered frontend for [trait.hagicode.com](https://trait.hagicode.com), providing a searchable agent catalog with crawlable canonical detail pages. The catalog snapshot lives at `src/data/generated/agent-catalog.json`.

## Working Directory

Run commands from `repos/trait/`.

## Key Commands

```bash
npm install
npm run dev
npm run build
npm test
npm run sync:agents
npm run seo:check
npm run i18n:check
```

## Key Paths

- `src/pages/`: Astro routes (home, catalog, detail pages)
- `src/components/`: React islands for search, filters, overlays
- `src/data/generated/agent-catalog.json`: catalog snapshot
- `scripts/`: agent sync, SEO validation, site snapshot sync

## Agent Guidelines

- Primary content renders as static HTML via Astro; use React islands only for interactivity.
- SEO metadata, JSON-LD, `robots.txt`, and `sitemap.xml` are generated during build from the catalog snapshot.
- Run `npm run sync:agents` to refresh the catalog from tracked sources before building.
- Run `npm run seo:check` after build to validate SEO metadata.

## References

- `README.md`
- `scripts/agent-sources.mjs`

---
> Source: [HagiCode-org/trait](https://github.com/HagiCode-org/trait) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
