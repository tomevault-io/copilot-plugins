## skillrush-town

> This repository is maintained through the project Skill:

# Agent Instructions

This repository is maintained through the project Skill:

`skills/skillrush-town/SKILL.md`

Before changing data ingestion, publishing, GitHub Pages, reports, or tests, read:

- `skills/skillrush-town/SKILL.md`
- `skills/skillrush-town/references/source-contract.md`
- `skills/skillrush-town/references/source-adapter-pattern.md`
- `skills/skillrush-town/references/publishing.md`
- `scripts/clawhub_daily.py`
- `README.md`

## Source Contract

Do not change the ClawHub Top100 source semantics unless the source contract is updated.

For non-ClawHub sources such as changelogs, model leaderboards, or release feeds, read `skills/skillrush-town/references/source-adapter-pattern.md` first and create a source-specific contract before implementation.

The canonical ranking source is:

- `POST https://wry-manatee-359.convex.cloud/api/query`
- Convex path: `skills:listPublicPageV4`
- args: `sort=downloads`, `dir=desc`, `nonSuspiciousOnly=true`, `highlightedOnly=false`, `numItems=25`
- pagination: follow `nextCursor` for 4 pages to build Top100

## Required Headless Validation

These checks must work without browser automation:

```bash
python -m py_compile scripts/clawhub_daily.py
python -m pytest -q
```

For a live ingestion check, use a temporary data directory:

```bash
TMP=$(mktemp -d)
python scripts/clawhub_daily.py --date 2026-05-04 --data-dir "$TMP/data"
```

## Optional Browser / Manual Page Check

Only run this when a browser is available:

```bash
python3 -m http.server 8093
```

Open:

```text
http://127.0.0.1:8093/?date=2026-05-04
```

---
> Source: [LearnPrompt/skillrush-town](https://github.com/LearnPrompt/skillrush-town) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
