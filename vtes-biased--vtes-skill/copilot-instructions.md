## vtes-skill

> This repo is the **source of truth for the `/vtes` agent skill** — a self-contained VTES

# CLAUDE.md

This repo is the **source of truth for the `/vtes` agent skill** — a self-contained VTES
(Vampire: The Eternal Struggle) expert skill. It is installed by symlinking the repo into
`~/.claude/skills/vtes`, so editing here edits the live skill. The goal: make an agent as
competent at VTES as an expert player — judge-level on rules, calibrated-analyst-level on
strategy.

## What is what

- **`SKILL.md`** — the always-loaded router and operating frame. Keep it lean: golden rules,
  condensed frame, routing table, data endpoints. Anything longer belongs in `references/`.
- **`references/rules/`**, **`references/strategy/`** and **`references/community/`** (the
  lower community tier — see `wiki/design.md` decision 6) — the knowledge. Two kinds of file
  live here, and the distinction governs every edit:
  - **Verbatim-carried** (most files): copied from an upstream repo, mapped in `SOURCES.md`.
    **Never hand-edit these** — fix the upstream and resync, or you fork silently. The one
    sanctioned local delta is recorded per-file in `SOURCES.md`.
  - **Synthesized** (`references/strategy/theory.md`, `SKILL.md`): curated in this repo. Upstream
    changes don't merge into them mechanically — an upstream diff is a *prompt* to revisit the
    synthesis, a curation decision.
- **`SOURCES.md`** — the provenance map: every verbatim file → upstream path, commit, copied-at
  date, and the dedup/drop decisions. Update it on every sync.
- **`scripts/`** — the data layer: `fetch_data.py` (KRCG v4 snapshot + usenet archives into
  gitignored `data/`), `query.py` (card/rulings/deck/search/rates/company against the snapshot).
  Stdlib-only by design — they must run anywhere.
- **`wiki/`** — standing knowledge: design decisions and rationale. Read `wiki/design.md` before
  changing the architecture.
- **`BOARD.md`** — what must change, in priority order. The goal is zero; completion is deletion.
- **`calibration/`** — the owner-graded corpus (`corpus/`, format in its README), the distilled
  `LESSONS.md` (synthesized here; the carried codex `calibration-lessons.md` stays frozen), and
  the regression set (`regression: yes` entries). Filled by the `/calibrate` skill.

## Standing constraints

- **Harness-neutral knowledge.** Everything under `references/` is plain Markdown/JSON with no
  Claude-Code-isms, and `scripts/` is stdlib Python, so the knowledge and data layers can be
  reused by other harnesses (a public bot, another agent). Claude-Code-specific files are
  `SKILL.md` and this repo's own `.claude/` harness — nothing else.
- **Self-contained.** No file references into other repos. Network grounding (api.krcg.org,
  static.krcg.org) is first-class and encouraged; cross-repo file paths are forbidden.
- **Grounding discipline applies to maintenance too.** When editing knowledge files, verify
  claims against the rulebook/rulings/API — never from memory.
- **Bulk data is not committed.** Everything under `data/` is gitignored — the KRCG snapshots
  (regenerable from static.krcg.org), their freshness stamp, and the usenet archives (durably
  hosted as the `usenet-archives` release on this repo). Only the scripts are committed.

## Maintenance loops

- **Sync**: the `/sync` skill (`.claude/skills/sync/`) wraps `scripts/sync.py` — `status`
  refreshes shallow clones of the upstreams into `data/upstreams/` and diffs every manifest
  entry; `apply` lands mechanical changes and refreshes the snapshots (`scripts/sync_snapshots/`)
  behind curated and watch entries. Every diff is adjudicated as ingress before applying; the
  skill carries the ordering rules (merge/assess first, apply second).
  Upstreams: `vtes-biased/vtes-advanced-rules` (+ its submodules) and
  `lionel-panhaleux/codex-of-the-damned`; the pre-repo skill files are owned here, no upstream.
- **Calibration**: the `/calibrate` skill (`.claude/skills/calibrate/`) runs graded sessions
  with the owner — answers recorded verbatim in `calibration/corpus/` with the grade and
  correction, generalizable corrections distilled into `calibration/LESSONS.md` — and regression
  re-runs after skill revisions (fresh, uncontaminated answers to every `regression: yes`
  question, drift reported for owner adjudication). Community feedback, when it arrives, is
  *candidate* entries — the owner remains the judge.

## Commits

Trunk-based: straight to `main`, fast-forward, no feature branches. Describe the change itself.

---
> Source: [vtes-biased/vtes-skill](https://github.com/vtes-biased/vtes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
