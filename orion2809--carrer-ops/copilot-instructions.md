## carrer-ops

> > AI-powered job search pipeline: offer evaluation, CV generation, portal scanning, application tracking.

# Career-Ops — Project Guidelines

> AI-powered job search pipeline: offer evaluation, CV generation, portal scanning, application tracking.

## Architecture

- **Data files**: Markdown tables (applications.md), YAML config, TSV batch files
- **Scripts**: Node.js `.mjs` (ES Modules) — PDF generation, tracker merge, dedup, normalization
- **PDF Engine**: Playwright (Chromium headless) via `node generate-pdf.mjs`
- **Dashboard TUI**: Go (Bubble Tea + Lipgloss) in `dashboard/`
- **Templates**: HTML (cv-template.html), YAML (states.yml, portals)

## Data Contract (CRITICAL)

Two layers — never mix them:

| Layer | Files | Rule |
|-------|-------|------|
| **User** | cv.md, config/profile.yml, modes/_profile.md, article-digest.md, portals.yml, data/*, reports/*, output/*, interview-prep/* | NEVER auto-updated |
| **System** | modes/_shared.md, all mode files, *.mjs scripts, dashboard/*, templates/*, batch/*, docs/* | Safe to auto-update |

**THE RULE**: User customizations go in `modes/_profile.md` or `config/profile.yml`. NEVER edit system files for user-specific content.

## Commands

```bash
node verify-pipeline.mjs        # Health check (7 checks)
node merge-tracker.mjs           # Merge batch TSVs → applications.md
node dedup-tracker.mjs           # Remove duplicate entries
node normalize-statuses.mjs      # Map status aliases to canonical
node cv-sync-check.mjs           # Validate setup consistency
node generate-pdf.mjs <in> <out> # HTML → PDF via Playwright
node update-system.mjs check     # Check for updates
```

## Conventions

- Report numbering: sequential 3-digit zero-padded (`001`, `002`, ...)
- Report naming: `{###}-{company-slug}-{YYYY-MM-DD}.md`
- Canonical states: see `templates/states.yml` (Evaluated, Applied, Responded, Interview, Offer, Rejected, Discarded, SKIP)
- NEVER edit `data/applications.md` to ADD entries — write TSV to `batch/tracker-additions/` and use `merge-tracker.mjs`
- YES you can edit `data/applications.md` to UPDATE existing entries
- All reports MUST include `**URL:**` in the header
- NEVER hardcode metrics — read from cv.md + article-digest.md at evaluation time
- NEVER invent experience or skills
- NEVER submit applications without user reviewing first

## Ethical Guidelines

- Quality over quantity — 5 targeted applications > 50 generic blasts
- Strongly discourage applications with score < 4.0
- Respect recruiter time — only send what's worth reading
- Fill forms but STOP before clicking Submit/Send/Apply

## Sources of Truth (read order)

1. `cv.md` — canonical CV
2. `article-digest.md` — detailed proof points (optional)
3. `config/profile.yml` — candidate identity
4. `modes/_profile.md` — user customizations (overrides _shared.md)

## Key Folders

| Path | Purpose |
|------|---------|
| `modes/` | Agent instruction files (scoring, evaluation, scanning) |
| `data/` | User's application tracker, pipeline inbox, scan history |
| `reports/` | Evaluation reports (markdown) |
| `output/` | Generated PDFs |
| `batch/` | Batch processing state, logs, tracker additions |
| `templates/` | HTML template, states, portal examples |
| `config/` | User profile (YAML) |
| `dashboard/` | Go TUI application |
| `fonts/` | Space Grotesk + DM Sans |

## Scoring System

6-block evaluation (A-F), each 1-5 scale:
- **A**: Role Summary (archetype, domain, function, seniority)
- **B**: CV Match (JD requirements vs CV proof points)
- **C**: Level & Strategy (seniority positioning)
- **D**: Comp & Demand (market salary data)
- **E**: CV Personalization (top changes for match)
- **F**: Interview Prep (STAR+R stories)

Score interpretation: 4.5+ strong match, 4.0-4.4 good, 3.5-3.9 decent, <3.5 recommend against.

---
> Source: [ORION2809/carrer-ops](https://github.com/ORION2809/carrer-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
