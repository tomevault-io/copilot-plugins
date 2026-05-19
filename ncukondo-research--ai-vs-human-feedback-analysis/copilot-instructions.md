## ai-vs-human-feedback-analysis

> This repository contains the manuscript, analysis code, and submission materials for a mixed-methods study comparing AI-generated feedback with human supervisor feedback on medical students' clinical clerkship logs. The manuscript is submitted to JMIR Medical Education.

# CLAUDE.md

## Project Overview

This repository contains the manuscript, analysis code, and submission materials for a mixed-methods study comparing AI-generated feedback with human supervisor feedback on medical students' clinical clerkship logs. The manuscript is submitted to JMIR Medical Education.

## Repository Structure

- `src/manuscript.qmd` — Main Quarto manuscript (renders to DOCX via `compile.sh`)
- `src/analysis/` — R/Python analysis scripts (quantitative, qualitative, integration)
- `data/` — Raw and processed datasets
- `submission/JMIR-Medical-Education/` — Submission history and reviewer correspondence
  - `2_review/one_by_one_reply.md` — Point-by-point reply to reviewers
- `library.json` — Exported subset of references cited in the manuscript (not the full library)
- `_quarto.yml` — Quarto project configuration
- `JMIR.csl` — Citation style for the journal

## Citation and Reference Management

References are managed with the `ref` CLI (reference-manager). The full reference library is maintained externally and accessed via `ref`. The `library.json` in this repository is an export containing only the references actually cited in the manuscript.

### In the manuscript (`manuscript.qmd`)

Use Pandoc citation syntax: `@pandoc-key` or `[@pandoc-key]`. Pandoc resolves these automatically during rendering.

Example: `[@Hansen2024-fe; @Ende1983-yv]`

### In reviewer replies and other non-rendered documents

Do NOT use `@pandoc-key` syntax. These files are not processed by Pandoc, so citation keys will not be expanded. Instead, write out citations manually (e.g., "Hansen et al., 2024" or use numbered references).

### Adding new references

1. Use `ref search <query>` to check if the reference already exists in the full library.
2. If the reference is not found and a literature search is needed, use the `search-hub` CLI to search academic databases.
3. Add the reference with `ref add`.

**Important:** When using `ref` or `search-hub` commands for the first time, always run the command with `--help` first to confirm available subcommands and options before executing.

## Writing Style

The manuscript and reviewer replies must follow academic journal conventions:

- Do not use em dashes. Use commas, semicolons, parentheses, or restructure the sentence instead.
- Do not use bold or italic formatting for emphasis in the manuscript body or reviewer replies. However, italic is required for statistical symbols per JMIR convention (e.g., *P*<.001, Cohen *d*, *F*-test, *W*).
- Do not use bullet points or numbered lists in the manuscript body. Use continuous prose. (Bullet points are acceptable in reviewer reply documents where the format calls for them.)
- Write in a natural, formal academic register appropriate for a medical education journal.

---
> Source: [ncukondo-research/ai-vs-human-feedback-analysis](https://github.com/ncukondo-research/ai-vs-human-feedback-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
