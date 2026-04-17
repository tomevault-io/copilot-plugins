## mlstack

> ├── plan-science-review/   # PI-mode analysis plan review

# mlstack development

## Project structure

```
mlstack/
├── plan-science-review/   # PI-mode analysis plan review
├── plan-stats-review/     # Biostatistician methods review
├── plan-ml-review/        # ML Architect modeling review
├── review-notebook/       # Pre-submission notebook review
├── eda/                   # Systematic exploratory data analysis
├── feature-eng/           # Feature engineering with leakage detection
├── model-critique/        # Adversarial model evaluation
├── review-perf/           # ML Performance Engineer review
├── retro-analysis/        # Analytics team retrospective
├── ship-analysis/         # Package and ship reproducible analysis
├── setup                  # Setup script (registers skills, updates CLAUDE.md / AGENTS.md)
├── SKILL.md               # Top-level skill (Claude discovers this)
└── README.md              # Project overview
```

## Setup

Run `./setup` to register skills with Claude Code and/or Codex and update instruction files.
Run `./setup --help` for options. The script dynamically discovers skills from SKILL.md
frontmatter — no hardcoded skill list to maintain when adding new skills.

## mlstack skills

Slash commands that switch Claude Code into specialist modes for data analysis:

- `/plan-science-review` — PI-mode: challenge the research question, map causal structure, audit assumptions before anyone touches data.
- `/plan-stats-review` — Biostatistician-mode: lock in methodology, power analysis, validation strategy, multiple comparison corrections.
- `/plan-ml-review` — ML Architect-mode: challenge representation strategy, architecture selection, training dynamics, compute-performance tradeoffs. The person who asks "why are you flattening images to feature vectors?" and "why are you training a ViT-L on 500 samples?"
- `/review-notebook` — Paranoid reviewer: find data leakage, undisclosed multiple comparisons, assumption violations, p-hacking patterns.
- `/eda` — Senior analyst: systematic exploratory analysis producing structured Jupyter notebooks with distribution profiles, missingness maps, multicollinearity checks.
- `/feature-eng` — ML engineer: domain-aware feature creation with explicit rationale, leakage detection, and selection justification.
- `/model-critique` — Adversarial evaluator: challenge every modeling choice. "Was nested CV actually the best tool here?" "Your pipeline assumed normality but these features violate that assumption."
- `/review-perf` — ML Performance Engineer: profile and optimize training/inference code. GPU utilization, dataloader throughput, mixed precision, memory footprint, unnecessary computation. Every suggestion includes estimated speedup and concrete code fix.
- `/retro-analysis` — Analytics manager: team retrospective tracking notebook quality, methodology rigor, reproducibility, and insight yield.
- `/ship-analysis` — Release analyst: freeze environment, validate reproducibility, generate executive summary, archive artifacts.

## Skill interaction patterns

The skills are designed to be used in sequence (but any can be used standalone):

```
  /plan-science-review  →  /plan-stats-review  →  /plan-ml-review  →  /eda  →  /feature-eng
         ↓                        ↓                       ↓              ↓          ↓
    [Is this the            [Is the method       [Is the arch.    [What does  [Features
     right question?]        sound?]              right?]          the data    ready]
                                                                   look like?]
                                                      ↓
                              /model-critique  ←  [modeling]
                                    ↓
                              /review-perf  →  /review-notebook  →  /ship-analysis  →  /retro-analysis
```

## Cross-agent critique

A key feature of mlstack: agents can and should critique each other's recommendations.

Examples:
- `/model-critique` might say: "The `/plan-stats-review` recommended nested CV, but with n=50,000 that's 100x more computation for negligible improvement."
- `/plan-ml-review` might say: "The `/feature-eng` is hand-engineering spectral features from these waveforms, but a 1D-CNN on raw signals would learn better representations with less effort."
- `/plan-ml-review` might say: "The `/plan-stats-review` recommends checking normality assumptions, but we're using a tree-based model — distributional assumptions don't apply here."
- `/review-notebook` might say: "The `/eda` notebook identified skewed features but the modeling notebook didn't address them."
- `/review-perf` might say: "The `/model-critique` validated results from a training run where the GPU was 40% idle — the complex model may have been handicapped by a dataloader bottleneck, making the model comparison unfair."
- `/review-perf` might say: "The `/feature-eng` preprocessing pipeline runs entirely on CPU through PIL — moving to Kornia on GPU would cut preprocessing time by 5x."
- `/plan-science-review` might say: "The `/feature-eng` created an interaction term that has no domain justification."

This ensures no single agent's blind spots persist through the pipeline.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tim-krausz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
