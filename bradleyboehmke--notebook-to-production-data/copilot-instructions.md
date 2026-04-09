## notebook-to-production-data

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a **Quarto/RevealJS slide deck** for the "Applied Workshops" series. The presentation teaches undergraduate and graduate students about **DataOps principles** and demonstrates how to transform a simple Jupyter notebook into a production-ready data pipeline.

The workshop uses the **Rossmann forecasting project** as a practical example, covering validation gates, modular code structure, feature engineering, and data versioning with DVC.

## Related Repositories

This project builds upon two existing repositories that serve as essential references:

### 1. Portfolio Workshop Template
**Path:** `../portfolio-via-github`
**Purpose:** Structural and stylistic template for workshop presentations

This repository contains a previous workshop on "Building Your Portfolio with Git & GitHub." The current DataOps presentation should **mirror its structure, styling, and pedagogical approach**:
- RevealJS theme and configuration (simple theme, fade transitions)
- Slide layout patterns (dark backgrounds `#43464B`, callout usage)
- Directory structure: `workshop/slides/*.qmd` with `assets/` subdirectory
- Speaker notes format (`::: {.notes}` blocks)
- Educational philosophy: momentum over mastery, practical over theoretical

### 2. Rossmann Forecasting DataOps Pipeline
**Path:** `/Users/b294776/Desktop/Projects/rossmann-forecasting`
**Purpose:** The actual implementation referenced in the workshop

This is the **live example project** that demonstrates DataOps principles. The presentation draws from its three-stage maturity model:

- **Stage 1** (`1-initial-exploration` branch): Notebook-only workflow showing the "before" state
- **Stage 2** (`2-research-mature` branch): Modular code structure with `src/` directory
- **Stage 3** (`main` branch): Full production MLOps with monitoring and CI/CD

**Key Pipeline Components** (referenced in slides):
- `scripts/dataops_workflow.sh` — Automated 6-step DataOps workflow
- `src/data/` — Data processing and validation modules
- `src/features/` — Feature engineering modules
- Data validation stages: raw → processed → features
- DVC integration for data versioning

## Specification Reference

All content requirements, structure, and styling guidelines are defined in [spec.md](spec.md). Always refer to this specification when creating or modifying slides to ensure alignment with workshop objectives.

## Project Structure

**Expected Directory Layout** (mirroring portfolio-via-github):

```
.
├── workshop/
│   ├── slides/
│   │   ├── dataops-pipeline.qmd      # Main Quarto presentation file
│   │   ├── dataops-pipeline.html     # Rendered output
│   │   ├── assets/
│   │   │   └── images/               # Custom images and diagrams
│   │   └── _extensions/              # RevealJS plugins (if needed)
│   └── templates/                    # Optional: Student exercise templates
├── spec.md                           # Complete workshop specification
└── CLAUDE.md                         # This file
```

**Note:** The portfolio-via-github repo uses `workshop/slides/` structure. Follow this convention for consistency across the Applied Workshops series.

## Development Commands

### Render Slides
Compile the Quarto file to HTML (following portfolio-via-github structure):
```bash
quarto render workshop/slides/dataops-pipeline.qmd
```

Output: `workshop/slides/dataops-pipeline.html`

### Preview During Development
Launch a live preview server that auto-reloads on changes:
```bash
quarto preview workshop/slides/dataops-pipeline.qmd
```

### Check Quarto Version
```bash
quarto --version
```

System is running Quarto 1.7.31.

### Reference Portfolio Workshop Locally
View the portfolio-via-github slides for structural patterns:
```bash
open ../portfolio-via-github/workshop/slides/portfolio-github.html
```

## Architecture & Design Principles

### Slide Deck Configuration
- **Theme**: RevealJS "simple" theme
- **Transition**: Fade
- **Plugins**: `appearance` and `highlight-text` for callouts and emphasis
- **Slide Numbers**: Enabled
- **Footer**: Links to DataOps resources (Rossmann GitHub repository)

### Visual Style
- Follow the visual conventions of the portfolio workshop slides
- Use dark backgrounds (`#43464B`) for main slides
- Bright accent colors for emphasis
- Callout boxes (`.callout-tip`, `.callout-warning`, etc.) for key points
- Mermaid diagrams for pipeline flows and architecture visualizations

### Content Structure
The deck follows a pedagogical flow:
1. **Welcome/Title** → Agenda
2. **Problem Introduction** → Simple notebook demo with limitations
3. **Solution Framework** → DataOps principles and benefits
4. **Technical Deep Dive** → Pipeline architecture and implementation steps
5. **Practice** → Student exercise and portfolio examples
6. **Wrap-up** → Key takeaways and next steps

### Presenter Notes
Use `::: {.notes}` blocks for speaking guidance and additional context. These appear in presenter view but not in student-facing slides.

## External References

### Rossmann DataOps Documentation
The workshop references the Rossmann forecasting project's DataOps implementation:
- **Repository:** [bradleyboehmke/rossmann-forecasting](https://github.com/bradleyboehmke/rossmann-forecasting)
- **Local Path:** `/Users/b294776/Desktop/Projects/rossmann-forecasting`
- **Branch `1-initial-exploration`:** Contains the baseline notebook showing "before" state
- **Main Branch:** Full production implementation with 3-stage maturity progression

### DataOps Workflow Details (from rossmann-forecasting)

The automated pipeline (`scripts/dataops_workflow.sh`) implements these 6 stages:

1. **Validate Raw Data**
   - Command: `python src/data/validate_data.py --stage raw --fail-on-error`
   - Checks: Schema compliance, data types, value ranges, null handling
   - Files: `data/raw/train.csv`, `data/raw/store.csv`

2. **Process Raw Data**
   - Command: `python -m src.data.make_dataset`
   - Actions: Merge datasets, convert types, handle missing values
   - Output: `data/processed/` (Parquet format for efficiency)

3. **Validate Processed Data**
   - Command: `python src/data/validate_data.py --stage processed --fail-on-error`
   - Ensures: Processing didn't introduce errors or data corruption

4. **Build Standard Features**
   - Command: `python -m src.features.build_features`
   - Creates: Calendar features, promo features, competition features, lag/rolling aggregates
   - Location: `src/features/` module with reusable functions

5. **Validate Features**
   - Command: `python src/data/validate_data.py --stage features --fail-on-error`
   - Detects: NaN values, infinite values, data leakage from future information

6. **Version Control with DVC**
   - Command: `dvc add data/processed/` and `dvc push`
   - Purpose: Track large data files separately from Git, enable reproducibility
   - Benefits: Rollback to previous data versions, share data across team

### Module Structure in rossmann-forecasting
```
src/
├── data/
│   ├── make_dataset.py      # Data processing pipeline
│   └── validate_data.py     # Great Expectations validation
├── features/
│   └── build_features.py    # Feature engineering functions
├── models/
│   ├── train_ensemble.py    # Model training
│   └── predict.py           # Production predictions
└── utils/                   # Shared utilities
```

This modular structure demonstrates the **key principle**: Move code from notebooks into reusable, testable modules under `src/`.

## Working with Quarto/RevealJS

### Slide Backgrounds (from portfolio-via-github)
Main section slides use dark background:
```markdown
# Main Section Title {background="#43464B"}
```

Image backgrounds for visual impact:
```markdown
## Slide Title {background-image="assets/images/photo.jpg" background-size="cover" background-position="center"}
```

### Adding Mermaid Diagrams
The Rossmann project README contains an excellent 3-stage maturity diagram. Embed similar diagrams:
````markdown
```{mermaid}
graph LR
    A[Raw Data] --> B[Validate]
    B --> C[Process]
    C --> D[Validate]
    D --> E[Features]
    E --> F[Validate]
```
````

### Creating Callouts
```markdown
::: {.callout-tip}
## Today's Goal
Walk out with the foundation of your portfolio started and a clear plan to keep building.
:::
```

Available types: `tip`, `warning`, `important`, `note`, `caution`

### Multi-Column Layouts (from portfolio-via-github pattern)
```markdown
::: {.columns}
::: {.column width="70%"}
Main content here
:::
::: {.column width="30%"}
![](assets/images/diagram.png){width=80%}
:::
:::
```

### Speaker Notes Format
```markdown
::: {.notes}
Share additional context here. These notes appear in presenter view but not in student-facing slides. Use this for delivery guidance, timing cues, and detailed explanations.
:::
```

## Educational Context & Pedagogy

This workshop is part of the **Applied Workshops Series**, which bridges the gap between classroom theory and industry practice. Following the pedagogical approach established in portfolio-via-github:

### Teaching Philosophy (Consistent Across Series)
- **Momentum over mastery:** Get students practicing quickly rather than overwhelming with theory
- **Practical over perfect:** Emphasize "good enough" solutions that work
- **Hands-on over lecture:** Minimize slide text, maximize demonstration and practice time
- **Inspire over instruct:** Show real examples (like Jeff Whitcomb's Yahoo Finance pipeline) to motivate

### Student Background Assumptions
- **Familiar with:** Jupyter notebooks, basic Python, exploratory data analysis
- **Some exposure to:** Git/GitHub (basic commits), data cleaning workflows
- **Likely new to:** Production pipelines, automated validation, data versioning (DVC), modular code architecture
- **Goal:** Bridge the gap from "it works on my laptop" to "it runs reliably in production"

### Content Guidelines
- Use speaker notes (`::: {.notes}`) for detailed explanations, NOT slide text
- Keep slides visual and concise — bullet points, not paragraphs
- Include callouts for key concepts and tips
- Provide specific, actionable examples rather than abstract principles
- End with clear next steps students can take immediately

### Workshop Structure (Per spec.md)
1. **Welcome/Title** → Set context and expectations
2. **Problem Introduction** → Notebook demo showing limitations
3. **Solution Framework** → DataOps principles overview
4. **Technical Deep Dive** → Pipeline walk-through with live code references
5. **Student Exercise** → Brainstorm personal pipeline projects
6. **Wrap-Up** → Key takeaways and resources for continued learning

## Quick Reference: Key Files to Consult

When building slides, frequently reference these files:

### Structural Templates
- [../portfolio-via-github/workshop/slides/portfolio-github.qmd](../portfolio-via-github/workshop/slides/portfolio-github.qmd) — Slide structure and styling patterns
- [../portfolio-via-github/spec.md](../portfolio-via-github/spec.md) — Pedagogical approach and content guidelines

### Technical Content Sources
- [/Users/b294776/Desktop/Projects/rossmann-forecasting/scripts/dataops_workflow.sh](/Users/b294776/Desktop/Projects/rossmann-forecasting/scripts/dataops_workflow.sh) — Complete automated pipeline script
- [/Users/b294776/Desktop/Projects/rossmann-forecasting/README.md](/Users/b294776/Desktop/Projects/rossmann-forecasting/README.md) — Three-stage maturity model and Mermaid diagram
- [/Users/b294776/Desktop/Projects/rossmann-forecasting/docs/dataops/](/Users/b294776/Desktop/Projects/rossmann-forecasting/docs/dataops/) — Detailed DataOps documentation

### This Project
- [spec.md](spec.md) — Complete content specification for this workshop

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bradleyboehmke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bradleyboehmke)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
