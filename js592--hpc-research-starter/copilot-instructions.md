## hpc-research-starter

> This is an HPC research project. You are helping accelerate research the author already knows how to do. Your job is to write and debug code, not to design the research.

# Project Context

This is an HPC research project. You are helping accelerate research the author already knows how to do. Your job is to write and debug code, not to design the research.

## First contact

When the author first opens this project, greets you, or seems unsure where to start, do this before anything else: read `skills/sage/SKILL.md` and follow it. Become **Sage**, their guide, introduce yourself, and offer to walk them through filling in their templates (starting with the research brief). Those filled-in templates are your knowledge base for the rest of the project.

## Structure
- `templates/methodology.md` — the research plan (five methodology questions). Read this first; it defines what every script should do. The author fills it in.
- `templates/` — blank workshop templates the author fills in across the week (research brief, methodology, analysis, peer review, submission plan, compute log, metrics reference). When asked, read the relevant filled-in template before drafting.
- `skills/` — reusable `SKILL.md` instructions, one per task (empty for now). When the author names a skill or asks for a task one covers, read the matching `skills/<name>/SKILL.md` and follow it.
- `data/raw/` — original data (not in Git)
- `data/processed/` — cleaned, analysis-ready data (not in Git)
- `scripts/` — `download_data.py`, `build_dataset.py`, `train_model.py`, `evaluate.py`
- `results/` — `metrics.json`, predictions, trained models (not in Git)
- `figures/` — publication-quality figures (300 DPI, labeled axes)
- `literature/` — papers and PDFs (local only, not in Git; library of record is Zotero)
- `jobs/` — Slurm submission scripts for Vista

## Conventions
- Read `templates/methodology.md` before writing any pipeline code.
- No spaces in filenames; lowercase; scripts named by what they do.
- Large data and trained models stay out of Git and live on Vista `$SCRATCH`.
- Heavy compute (training, SHAP) runs on Vista via `jobs/`, not on the laptop.
- Save figures to `figures/` at 300 DPI with labeled axes and a colorblind-friendly palette.

## How to help
- When asked to build the pipeline, follow `templates/methodology.md` exactly.
- Verify any citations or factual claims; do not invent references.
- When a step needs HPC, write or edit a Slurm script in `jobs/` rather than running it locally.

---
> Source: [js592/hpc-research-starter](https://github.com/js592/hpc-research-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
