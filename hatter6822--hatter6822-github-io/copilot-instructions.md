## hatter6822-github-io

> This repository is the static website for **seLe4n**, a formally verified microkernel written in Lean 4. It consists of two pages and a data pipeline:

# CLAUDE.md — seLe4n Website Project Guidance

## Project Overview

This repository is the static website for **seLe4n**, a formally verified microkernel written in Lean 4. It consists of two pages and a data pipeline:

- `index.html` — Marketing/overview landing page
- `map.html` — Interactive codebase map with dependency graph, theorem coupling, and declaration explorer
- `data/*.json` — Bundled snapshots consumed by the browser runtime
- `scripts/*.mjs` — Node.js tooling to regenerate and validate those snapshots

**Stack:** Pure HTML5 + CSS3 + Vanilla JavaScript ES6+ (no frameworks, no bundler). Node.js for offline tooling only.

**Website version:** `0.25.13`
**Lean toolchain target:** `4.28.0`

## Build and Validation Commands

### Required before every commit

```bash
# Parser and validation tests (all must pass, zero warnings)
node scripts/lib/lean-analysis.test.mjs
node scripts/lib/data-validation.test.mjs
node scripts/lib/map-runtime.test.mjs
node scripts/lib/map-toolbar.test.mjs

# Bundled data integrity
node scripts/validate-data.mjs

# JavaScript syntax checks
node --check assets/js/map.js
node --check assets/js/header-nav.js
node --check assets/js/site.js
node --check assets/js/i18n.js
node --check assets/js/background-pattern.js
node --check assets/js/theme-init.js
```

### Data refresh (when upstream seLe4n repo changes)

```bash
node scripts/sync-site-data.mjs
node scripts/sync-map-data.mjs
```

## Validation Tiers

| Tier | Scope | When to run |
|------|-------|-------------|
| 0 | JS syntax checks (`node --check`) | Every commit |
| 1 | Unit tests (`scripts/lib/*.test.mjs`) | Every commit |
| 2 | Data validation (`scripts/validate-data.mjs`) | Every commit, after data changes |
| 3 | Manual browser verification (desktop + mobile) | UI/layout changes |
| 4 | Playwright nav stability probe (`scripts/nav-stability-smoke.py`) | Navigation behavior changes |

Run at least Tiers 0-2 before any commit. Tier 3 for front-end changes. Tier 4 when touching navigation or scroll behavior.

## Large File Handling

Several files exceed 500 lines:

| File | Lines | Notes |
|------|-------|-------|
| `assets/js/map.js` | ~4,850 | Largest runtime; read in chunks of ≤500 lines |
| `assets/css/style.css` | ~1,833 | Global design system |
| `assets/css/map.css` | ~874 | Map-specific styles |
| `assets/js/background-pattern.js` | ~846 | WebGL shader; contains third-party noise code |
| `assets/js/site.js` | ~788 | Landing page runtime |
| `assets/js/header-nav.js` | ~738 | Shared navigation controller |

**Rules:**
- Never read an entire large file in one operation. Use offset/limit (≤500 lines per read).
- Use the Edit tool for modifications to existing files, never Write.
- Keep edits surgical — one logical change per Edit call.

## Key Architectural Conventions

### Runtime data strategy (local-first)

1. Load bundled `data/*.json` immediately
2. Hydrate from browser `localStorage` cache if newer
3. Attempt live refresh from GitHub APIs (with cooldown + jitter)
4. Fall back gracefully if network refresh fails

### Map data normalization

- `modules[]` array is the canonical source of graph nodes
- Legacy top-level maps (`moduleMap`, `importsFrom`, `moduleMeta`) are fallbacks only
- Branch-ref metadata keys (e.g. `main`) are excluded from module inventories
- Declaration-centric payloads (`modules[].declarations`) are projected into symbol buckets
- Reverse import edges (`importsTo`) are always rebuilt from `importsFrom`

### Security posture

Both HTML pages enforce:
- `Content-Security-Policy` (strict `default-src 'self'`)
- `Permissions-Policy` (geolocation, microphone, camera, payment, USB, browsing-topics disabled)
- `referrer` policy (`strict-origin-when-cross-origin`)
- `X-Content-Type-Options: nosniff`
- All external links hardened with `rel="noopener noreferrer"`

### Operations/Invariant split (upstream seLe4n convention)

The codebase map recognizes the Operations.lean/Invariant.lean pair pattern. Proof pairs are detected, scored, and visualized with assurance labels (linked/partial/local/none).

## File Ownership Quick Reference

| Change area | Primary file(s) |
|-------------|-----------------|
| Map graph behavior | `assets/js/map.js` |
| Map controls/layout | `map.html`, `assets/css/map.css` |
| Landing page metrics | `assets/js/site.js`, `index.html` |
| Navigation behavior | `assets/js/header-nav.js` |
| Theme switching | `assets/js/theme-init.js` |
| Internationalization | `assets/js/i18n.js`, `locales/*.json` |
| Background animation | `assets/js/background-pattern.js` |
| Lean parsing | `scripts/lib/lean-analysis.mjs` |
| Data validation | `scripts/lib/data-validation.mjs` |
| Global styles | `assets/css/style.css` |

## Documentation Sync Requirements

When making changes, keep these documents in sync:

- `README.md` — Project overview and workflow
- `CONTRIBUTING.md` — Required checks and checklists
- `docs/ARCHITECTURE.md` — System architecture decisions
- `docs/CODEBASE_MAP.md` — Map pipeline and runtime behavior
- `docs/DEVELOPER_GUIDE.md` — File-by-file orientation
- `docs/TESTING.md` — Testing matrix and manual verification

## Security Reporting

Any suspected CVE-worthy vulnerability discovered during work — whether in code logic, dependencies, build infrastructure, or specification gaps — must be reported immediately before continuing other work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hatter6822) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
