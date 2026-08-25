## eki-stamp-tracker

> Interactive Leaflet map of Japan for collecting eki stamps (駅スタンプ) and logging

# CLAUDE.md — Eki Stamp Tracker

Interactive Leaflet map of Japan for collecting eki stamps (駅スタンプ) and logging
ridden train-line sections. **Static files only: no build step, no npm, no bundler.**
`index.html` + native ES modules in `js/` + `css/app.css` + JSON/GeoJSON in `data/`.

The README is the operative manual (code map, data shapes, feature behaviour, regen
pipeline). `docs/` holds the design-of-record documents. Read the README section for
the area you're touching before editing.

## Skills — read the matching one BEFORE working in its area

| If the task involves… | Read |
|---|---|
| Running the app, verifying a change, headless/Playwright testing, `window.__eki` | `.claude/skills/run-and-verify` |
| Ride overlays, line highlighting, stitching/routing, gaps, Shinkansen paths, `js/geometry.js`, `js/line-geometry.js` | `.claude/skills/geometry-pipeline` |
| The gap audit, CI `MAX_GAPS` baseline, a new/changed gap count | `.claude/skills/ride-gap-audit` |
| `data/*` files, scrapers, `scripts/build_*`, regenerating or validating data | `.claude/skills/data-pipeline` |
| Stamps/rides persistence, localStorage, Gist sync, sessions, import/export, tokens | `.claude/skills/state-and-sync` |
| Adding/changing any feature or UI, new modules, popups, names, styling | `.claude/skills/conventions` |
| Shipping to main, `APP_VERSION`, CI failures, README updates | `.claude/skills/release-checklist` |

## Hard rules (history-backed; details + whys live in the skills)

1. **NEVER embed or share credentials.** A previous version shipped an obfuscated
   GitHub PAT — full audit fiasco (`docs/AUDIT-2026-07.md` Block 0). Per-user
   `gist`-scope token in localStorage only.
2. **NEVER lose user progress.** Local-first: every stamps/rides mutation mirrors to
   localStorage; session load MERGES local unsynced progress, never replaces.
3. **NEVER invent straight track.** Ride overlays only slice existing drawn geometry;
   a data gap stays a gap.
4. **NEVER join stations↔track geometry by name** — measured dead end
   (`docs/HANDOVER-line-highlight.md`). Geometry picks what to colour; ekidata
   ordered groups pick which stations.
5. **NEVER introduce a build step/toolchain.** New code = new ES module in `js/`
   matching the existing header style; `js/geometry.js` stays pure (unit-testable
   via `node --test tests/*.test.mjs`).
6. **Bump `APP_VERSION`** (`js/config.js`) on every merge to main — it keys both
   IndexedDB caches; forgetting it leaves users on stale data for up to 7 days.
7. **`esc()` every value interpolated into HTML** — names come from a scraped
   external site (stored-XSS history).
8. **Use curated display names, never ekidata romaji**; respect `state.lang`.
9. **`window.__eki`** (`js/main.js`) is the public tooling contract — extend it,
   never reshape it.
10. The 15-gap audit baseline and the "known hard cases" (central-Tokyo relabelled
    track, rapid/local parallel alignments, 成田線 branch order) are **documented
    non-bugs** — do not "fix" them casually; see the geometry-pipeline skill.

## Quick commands

```bash
python3 -m http.server 8000          # serve (required; file:// cannot load ES modules)
node --test tests/*.test.mjs         # pure-geometry unit tests (the bare "tests/" dir form fails on Node 22)
python3 scripts/check_data.py        # structural data sanity
# full regression gate — needs a served repo + playwright setup first;
# follow .claude/skills/ride-gap-audit for the working sandbox recipe:
BASE_URL=http://127.0.0.1:8097 MAX_GAPS=15 node scripts/audit-ride-gaps.mjs
```

---
> Source: [txmnzia/eki-stamp-tracker](https://github.com/txmnzia/eki-stamp-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
