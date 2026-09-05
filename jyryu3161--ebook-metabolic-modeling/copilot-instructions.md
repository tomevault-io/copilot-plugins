## ebook-metabolic-modeling

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A book, not software. It is a Korean-language undergraduate/graduate textbook on **genome-scale metabolic modeling (GEM)**, authored in Markdown and published with GitBook. There is no compile/build step for the prose — GitBook renders `SUMMARY.md` and the chapter files directly. The only executable code is two Python helper scripts and one static interactive page.

Because the deliverable is prose, the "correctness" gate is editorial consistency, and it is enforced mechanically (see below). Treat `docs/textbook-editorial-standard.md` as the authoritative spec — it is enforced, not aspirational.

## Commands

```bash
# Structural gate — run before considering any chapter change done.
python scripts/validate_textbook.py

# Regenerate the optimization figures (writes PNG+SVG to assets/figures/).
python scripts/generate_optimization_figures.py
```

There is no test suite, linter, or package manifest in the usual sense. `validate_textbook.py` **is** the check. It scans every `*.md` in the repo and fails on: malformed LaTeX escapes (`\qquad`, `\left\{` written without the leading backslash), a missing `textbook-completeness-supplement.md` link in `SUMMARY.md`, and any `chapter-{1..10}/README.md` that lacks its `interactive/index.html?chapter=N` embed.

Gotcha: because it uses `rglob('*.md')`, it also scans `.claude/skills/`. Writing a skill doc that mentions `\qquad`/`\left\{` as a bare literal will fail the gate — escape it.

Reference execution environment for any computed result: Python 3.10+, COBRApy 0.30.0, GLPK, the BiGG `e_coli_core` / `textbook` model. See `installation.md`.

## Repository structure that matters

- `chapter-{1..11}/` — each holds `README.md` (chapter intro), `01.md`…`NN.md` (numbered theory sections), `lab.md` (hands-on), `summary.md` (recap). ch10 and ch11 are the tutorial chapters; ch11 has no `lab.md`.
- `SUMMARY.md` — GitBook navigation. Adding/removing/renaming any chapter, section, or supplement **must** be mirrored here or the book's TOC drifts from the files.
- `docs/textbook-editorial-standard.md` — the editorial spec that everything is checked against.
- `.claude/skills/ebook-*` — five project skills (`ebook-prose`, `ebook-figure`, `ebook-citation`, `ebook-review`, `ebook-structure`) that encode the editorial standard for application. Use them when writing, reviewing, or restructuring content.
- `FIGURE_SOURCES.md` — the figure rights/reproducibility ledger. Every figure has a row (asset path, first-use location, generation conditions, concept source, rights, verification date). New or regenerated figures must be registered here.
- `glossary.md`, `landmark-papers.md` — link targets for terms and method papers; prose links to these rather than re-defining.
- `supplements/`, `workbook.md`, `textbook-completeness-supplement.md` — supplementary material reachable from `SUMMARY.md`.

## Two coupled subsystems to understand before editing

### Figures: two-location asset flow

`scripts/generate_optimization_figures.py` writes both PNG and SVG to `assets/figures/<stem>.{png,svg}` via a shared `save()` (transparent SVG + white-background PNG, fixed palette and `rcParams`). Prose does **not** reference `assets/figures/` — it references a **PNG copy** at `.gitbook/assets/<stem>.png` (as `../.gitbook/assets/<stem>.png`). After adding or regenerating a figure you must copy the PNG into `.gitbook/assets/` and add/update its `FIGURE_SOURCES.md` row.

Captions distinguish **schematic** (drawn from equations/coordinates, no solver) from **computed** (actual model run) figures, and this distinction must be stated in the caption. Self-made figures still cite a concept source: `저자 작성; 개념 근거: [저자 (연도)](DOI)`.

### Interactive page: hardcoded per-chapter coupling

`interactive/index.html` is a single self-contained page with a `?chapter=N` query param and **hardcoded visualizations for chapters 1–10 only** (functions `ch1`…`ch10`). Every `chapter-{1..10}/README.md` embeds it via GitBook `{% embed %}`, and `validate_textbook.py` enforces that embed link exists. Changing chapter numbering means updating both the embeds and this file.

## Editorial rules that cause the most churn

These are the non-obvious, enforced rules. The `ebook-prose` and `ebook-review` skills carry the full detail and the correct detection method.

- **Sentence endings are split by file type and never mixed in one file.** Theory (`README.md`, numbered sections, `summary.md`) uses academic 다체 (`~한다/이다`). Labs plus all of ch10/ch11 use 습니다체 (`~합니다/입니다`). This applies even to endings wrapped in bold/italic (`**확인한다**.` → `**확인합니다**.`) — bold-wrapped forms are the ones most often missed by naive search.
- **Code-block comments are noun phrases** (`# 성장률 계산`), never prose endings, regardless of file type.
- **Caption labels are Korean** — `그림 N.x`, `표 N.x`, summary tables `표 S.x` — with chapter-global numbering and no gaps. No `Figure`/`Table`.
- **Section subheadings**: noun-phrase or declarative both allowed; only rhetorical **question-form** titles (`왜 …하는가?`) are forbidden. Do not "fix" noun-phrase titles into sentences.
- **Answers are never exposed adjacent to the question.** Hints give solution direction only; final values/verdicts move to a `## 답안` section at the end of the chapter summary.
- **Computed results record provenance**: model ID+version, medium/bounds, objective, solver, tolerance. Model output is a conditional prediction, kept distinct from experimental observation.
- **Canonical term forms**: 대사물 (not 대사산물), 정상상태, 의사-정상상태 (hyphenated, matching `glossary.md`). Introduce English/abbreviation once on first use, then use one form consistently.

When measuring editorial conformance across the book, account for Markdown emphasis markers (`**`, `*`, `_`), inline/fenced code, and line-position variance (line-leading vs trailing comments) — bare regexes over raw text systematically under- or over-count.

---
> Source: [jyryu3161/ebook_metabolic_modeling](https://github.com/jyryu3161/ebook_metabolic_modeling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
