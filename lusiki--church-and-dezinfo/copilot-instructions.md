## church-and-dezinfo

> Research project analyzing **disinformation narratives in Croatian Catholic web media (2021-2024)**. Uses dictionary-based content analysis on ~443K web articles from the Determ monitoring platform. Three research papers examine (1) narrative framing, (2) structural proximity to disinformation ecosystems, and (3) engagement/amplification dynamics.

# CLAUDE.md - AI Assistant Instructions

## Project Overview

Research project analyzing **disinformation narratives in Croatian Catholic web media (2021-2024)**. Uses dictionary-based content analysis on ~443K web articles from the Determ monitoring platform. Three research papers examine (1) narrative framing, (2) structural proximity to disinformation ecosystems, and (3) engagement/amplification dynamics.

## Architecture

```
code/01_data_preparation.R  → data/*.rds     → papers/*.qmd
(ETL pipeline)                (4 corpora)      (3 papers + 1 analysis)
```

- **Language**: R (tidyverse style). Papers are Quarto (.qmd) with embedded R code chunks.
- **Paths**: All paths use `here::here()` relative to project root. No hardcoded paths.
- **Data**: `.rds` files are gitignored (1.8 GB total). Regenerate via `code/01_data_preparation.R`.

## Key Concepts

### Corpora (in data/)
| File | Description | ~Size |
|------|-------------|-------|
| `catholic_media_full_corpus.rds` | All web articles (443K rows) | 883 MB |
| `catholic_media_contested_corpus.rds` | Articles with any frame (311K) | 716 MB |
| `catholic_media_catholic_corpus.rds` | Catholic media only (93K) | 148 MB |
| `catholic_media_catholic_contested.rds` | Catholic + framed (71K) | 127 MB |

### Media Classification (7 types)
- **Catholic** (92,915 articles): Official Church (IKA, Glas Koncila), Catholic Radio, Catholic Portals (bitno.net), Catholic Aligned (narod.hr, dnevno.hr)
- **Conservative**: vecernji, hrt.hr, nacional
- **Liberal**: index.hr, telegram.hr, n1.hr
- **Tabloid**: 24sata, jutarnji
- **Regional**: slobodnadalmacija, novilist
- **Business**: poslovni, lider.hr
- **Other**: everything else (256K - largest category)

**Critical distinction**: "Official Church" vs "Catholic Aligned" is the most analytically important boundary. Paper 2 includes sensitivity analysis on this classification.

### 8 Narrative Frames (binary columns `frame_*`)
Detected via Croatian-language keyword dictionaries in `code/01_data_preparation.R` (lines 241-326):

| Frame | Croatian keywords (examples) | Role in NPI |
|-------|------------------------------|-------------|
| MORAL_DECAY | moralni pad, dekadencija, kultura smrti | - |
| FOREIGN_THREAT | nametanje, soros, globalizam | weight 1.5 |
| INSTITUTIONAL_DISTRUST | manipulacija, duboka država, propaganda | weight 1.5 |
| TRADITIONAL_VALUES | tradicija, obitelj, vjera, domovina | - |
| SOVEREIGNTY | suverenitet, neovisnost, volja naroda | - |
| CONSPIRACY | zavjera, big pharma, great reset | weight 2.0 |
| FAITH_DEFENCE | kršćanofobija, progon kršćana | - |
| MEDIA_CRITIQUE | mainstream mediji, fake news, pristranost | weight 1.0 |

### Narrative Proximity Index (NPI)
Composite index: `2*CONSPIRACY + 1.5*FOREIGN_THREAT + 1.5*INSTITUTIONAL_DISTRUST + 1*MEDIA_CRITIQUE`, normalized 0-100. Stored as `disinfo_alignment_norm`. Measures structural similarity to disinformation ecosystem narratives, NOT truth/falsity.

### 7 Actor Categories (binary columns `actor_*`)
CHURCH, GOVERNMENT, EU_ACTORS, NGO_CIVIL, SCIENTISTS, MEDIA_ACTORS, FAMILY_ORGS

### 6 Narrative Phases
1. COVID Peak (early 2021)
2. Post-Vaccine Debate (mid 2021 - Feb 2022)
3. Ukraine and Energy Crisis (Feb 2022 - Oct 2022)
4. Euro Adoption (Oct 2022 - Jan 2023)
5. Culture Wars Period (Jan 2023 - Jan 2024)
6. Election Run-up 2024 (Jan 2024+)

## Papers

### Paper 1: Framing (`papers/03_framing_paper.qmd`)
- **RQs**: Frame distribution across media types, narrative packages (co-occurrence), internal diversity of Catholic media
- **Methods**: Proportion tests, logistic regression (`frame ~ media_type + phase + log(word_count)`), conditional co-occurrence matrices
- **Key packages**: `broom`, `marginaleffects`

### Paper 2: Disinfo Index (`papers/04_disinfo_paper.qmd`)
- **RQs**: NPI differences across media, robustness to weight specs, narrative clusters, sensitivity to classification boundary
- **Methods**: 5 NPI weight specifications, hierarchical clustering (Ward), network analysis (`igraph`), reclassification sensitivity (Catholic Aligned → Conservative)
- **Key packages**: `igraph`, `ggdendro`

### Paper 3: Engagement (`papers/05_engagement_paper.qmd`)
- **RQs**: Which frames predict engagement, NPI-engagement relationship, Catholic vs other amplification patterns
- **Methods**: Negative binomial regression (`MASS::glm.nb`), separate Catholic/Other models, actor×frame engagement heatmaps
- **Key packages**: `MASS`, `marginaleffects`

## Common Patterns

### R code conventions
- Packages auto-installed if missing (for/require/install pattern)
- `options(dplyr.summarise.inform = FALSE, scipen = 999)`
- Theme: `theme_minimal(base_size = 12)` with bold titles, bottom legend
- Color palettes defined as named vectors (`frame_colors`, `media_colors`, `catholic_sub_colors`)
- Frame columns selected via `grep("^frame_", names(data), value = TRUE)`

### Known issues
- `02_analysis.qmd` references `sentiment_score`, `emotional_intensity`, `topic_*` columns that don't exist in the current data pipeline (sentiment/topic detection was removed in optimization). This file needs updating if rendered.
- `03_framing_paper.qmd` references `apa.csl` which is not in the repo. Remove or add the CSL file.
- Catholic Radio subcategory has only 4 articles - effectively negligible.

## Editing Guidelines

- **Croatian text** in papers is research content - preserve it. Don't translate unless asked.
- **Frame dictionaries** (lines 241-326 in `01_data_preparation.R`) are the analytical core. Changes here affect all downstream analyses.
- When adding new frames/actors: add to dictionaries → re-run pipeline → update all papers that reference frame/actor columns.
- The `here` package resolves paths relative to the project root (where `.here` or `.git` exists).

## Build Commands

```bash
# Render a specific paper
quarto render papers/03_framing_paper.qmd

# Render all papers
quarto render papers/

# Re-run data pipeline (requires source data)
Rscript code/01_data_preparation.R
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lusiki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lusiki)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
