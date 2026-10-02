## rsi-research

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Contains

A **self-improving-agent research repository** (`rsi-research`, public) — survey + tracked paper code only. It does not contain the 3C implementation or the patent work; both live in `../3c-meta` (github.com/beaugogh/3c-meta, private):

1. **Survey** (`docs/research-insights-draft.md`, Chinese) — 智能体自演进综述：self-improving systems classified by primary mutable object across three layers (Context / Workflow / Harness). `docs/research-insights.html` is the generated presentation. This is the currently active work.
2. **Paper notes** (`docs/202608*` + `_assets/`) — scraped copies and structured notes of the surveyed papers; source material for the survey, not project documentation.
3. **`repos/`** — git submodules: the surveyed papers' official code (ACE, ADAS, AFlow, AHE, MCE, Self-Harness; Autodata and ASPIRE have no public code) and `3c-meta` (the 3C framework + patent repo, also at `../3c-meta` as the working checkout).
4. **`scripts/md_to_html.py`** — regenerates the survey HTML from markdown.

## Commands

```bash
# Regenerate survey HTML (markdown is the source of truth)
.venv/bin/pip install markdown    # one-time; not in a requirements file
.venv/bin/python scripts/md_to_html.py
```

## Survey Conventions

- `docs/research-insights-draft.md` is the survey source of truth; regenerate `docs/research-insights.html` after editing. The script handles custom ```` ```flow ```` / ```` ```flow-tree ```` / ```` ```design-space ```` / table blocks — check it before inventing new block syntax.
- 3C references link to `../3c-meta` (e.g. `examples/security-report/docs/ARCHITECTURE.md`); the two 3C figures used by the survey (`docs/3c-dual-stage-arch.svg`, `docs/3c-workflow.svg`) stay local because the build script inlines local images as data URIs.
- The local skill `.claude/skills/research-insights-review/` defines how to review the survey (accuracy vs. derived vs. interpretation, comparison fairness, coverage, synthesis). Use it when asked to review the document.

## Notes

- `docs/temp_transcript_style_transfer/` is untracked in-progress research (SkillOpt/GEPA-adjacent, style-skill compilation from transcripts).
- The patent disclosure (Huawei confidential 交底书) now lives in `../3c-meta/patent/` — this repo no longer holds confidential material, but 3c-meta does; never make that repo public.
- `.venv/` is Python 3.13, kept for `md_to_html.py`; there is no other runtime code in this repo.

---
> Source: [beaugogh/rsi-research](https://github.com/beaugogh/rsi-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
