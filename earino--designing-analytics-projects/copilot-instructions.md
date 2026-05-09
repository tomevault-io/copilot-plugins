## designing-analytics-projects

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the course repository for **ECBS5228A "Designing Analytics Projects"**, a 1-credit intensive course for MS in Business Analytics students at Central European University. The course runs over 2 days (January 8 & 12, 2026) and prepares students for their mandatory capstone project.

## Course Structure

### Three Pillars

1. **Strategic Thinking**: Data science value extends beyond models in production (the "job titles are fragile, responsibilities are durable" narrative)
2. **Foundational Analyses**: 9 analyses structured along the customer journey
3. **Capstone Preparation**: Using the Analytics Project Brief framework for professional project design

### The 9 Foundational Analyses

| Phase | Analyses |
|-------|----------|
| **Acquisition** | Funnel, Channel Attribution, Campaign Effectiveness, CAC/LTV |
| **Retention** | Retention Analysis, Power User Analysis, Failure Analysis |
| **Growth** | Expansion & Monetization, Ecosystem Analysis |

Each analysis has a complete worked example in `templates/examples/`.

### The Analytics Project Brief

The centerpiece teaching tool is a 10-section project scoping framework:

1. Problem & Decision
2. Metrics (Primary + Counter-Metrics)
3. Stakeholder Map (Power-Interest Grid)
4. Methodology
5. Scope & Deliverables
6. Success & Decision Criteria
7. Timeline
8. Risks & Assumptions
9. Ethics & Privacy
10. Pre-Mortem

**Template:** `templates/analytics_project_brief.md`
**Examples:** `templates/examples/brief_01_*.md` through `brief_09_*.md`

## File Organization

```
/Users/earino/CEU/ECBS5228/
├── syllabus.md                    # Official course syllabus (authoritative)
├── CLAUDE.md                      # This file
├── README.md                      # GitHub-facing readme
├── LICENSE                        # MIT license
├── requirements.txt               # Python dependencies
│
├── templates/
│   ├── analytics_project_brief.md           # Blank Brief template
│   └── examples/                            # 9 worked examples
│       ├── brief_01_funnel_analysis.md      # Quickcart
│       ├── brief_02_channel_attribution.md  # DataDash
│       ├── brief_03_campaign_effectiveness.md # BrightMart
│       ├── brief_04_cac_ltv.md              # MindfulApp
│       ├── brief_05_retention_analysis.md   # SnapGram
│       ├── brief_06_power_user_analysis.md  # Streamflix
│       ├── brief_07_failure_analysis.md     # FindIt
│       ├── brief_08_expansion_monetization.md # NoteSpace
│       └── brief_09_ecosystem_analysis.md   # SocialSuite
│
├── slides/                        # Marp markdown slides
│   ├── block_00_syllabus_logistics.md
│   ├── block_01_analytics_project_brief.md
│   ├── block_02_acquisition_analyses.md
│   ├── block_03_retention_growth_analyses.md
│   ├── block_04_application_practice.md
│   ├── block_05_influence.md
│   └── block_06_capstone_closing.md
│
├── scenarios/                     # 18 unique student assignment scenarios
│   └── scenario_01_peakfit.md ... scenario_18_legaltech.md
│
├── weekly_writeups/
│   └── prompts.md                 # 6 weekly write-up prompts (Feb-Mar)
│
├── figures/
│   ├── figures.md                 # Designer briefs for all images
│   └── images/                    # Generated slide images (PNGs)
│
├── scripts/
│   └── check_overflow.py          # Slide overflow detection tool
│
└── old/                           # Deprecated content (gitignored)
```

### Gitignored Directories (Not in Public Repo)

These directories exist locally but are excluded from the GitHub repository:

| Directory | Contents |
|-----------|----------|
| `exam/` | Exam questions with answer key |
| `grading/` | Rubrics and scenario answer keys |
| `instructor_resources/` | Presenter guide, grading rubric, scenario "golden key" |
| `old/` | Deprecated files from earlier course iterations |

**Never commit these directories.** They contain answers students should not see.

## Common Development Tasks

### Building Slides

Slides use **Marp** (markdown slide generator).

```bash
# Build a single slide deck
cd slides/
marp --no-stdin block_02_acquisition_analyses.md --html -o block_02_acquisition_analyses.html

# Build all slides
for f in block_*.md; do
  marp --no-stdin "$f" --html -o "${f%.md}.html"
done
```

### Checking for Slide Overflow

After building, run the overflow checker:

```bash
python scripts/check_overflow.py
```

This uses headless Chromium to detect slides where content exceeds the 720px container height. Fix flagged slides before presenting.

**Requirements:**
```bash
pip install playwright
playwright install chromium
```

### Slide Generation Workflow

**Every time you modify slides:**

1. Build the HTML
2. Run `check_overflow.py`
3. Fix any overflowing slides
4. Verify visually by opening the HTML

## Assessment Structure

| Component | Weight | Due Date | Description |
|-----------|--------|----------|-------------|
| Scenario → Brief | 30% | Jan 16, 2026 | Complete Brief for assigned scenario |
| Pen & Paper Exam | 30% | Jan 27, 2026 | Closed book, one A4 cheat sheet |
| Weekly Write-ups | 30% | Jan 23 – Feb 27 | 6 practice exercises |
| Participation | 10% | During class | Engagement |

Each student receives one of 18 unique scenarios (`scenarios/scenario_*.md`).

## Key Course Concepts

### Counter-Metrics Framework

Counter-metrics prevent Goodhart's Law. The "What Breaks" 5-question framework:

1. What directly worsens?
2. What quality degrades?
3. Which user segments are harmed?
4. What long-term metrics suffer?
5. Which other teams' goals are threatened?

Students must identify 2-3 counter-metrics for every project, labeled as **Guardrail** (must not worsen) or **Tradeoff** (may worsen within bounds).

### Stakeholder Management

Uses the **Power-Interest Grid (Mendelow Matrix)**:

| | High Interest | Low Interest |
|---|---|---|
| **High Power** | Manage Closely | Keep Satisfied |
| **Low Power** | Keep Informed | Monitor |

Students also identify **Champions** (advocates) and **Blockers** (potential opponents) with mitigation strategies.

### Pre-Mortem Exercise

"It's 3 months from now and this project failed. What happened?"

Forces identification of non-obvious risks before they occur. Strong pre-mortems include:
- Correlation/causation confusion
- Stakeholder dynamics gone wrong
- Data quality issues discovered too late
- Scope creep

## Editing Guidelines

### Syllabus Changes

`syllabus.md` is the **authoritative student-facing document**. When editing:

- Maintain course code: `ECBS5228A (T2, AY 2025/26)`
- All dates reference January 2026 (Day 1: Jan 8, Day 2: Jan 12)
- Keep 6 blocks × 100 minutes structure
- Python-only program — never reference R

### Slide Standards

Each foundational analysis section should include:

1. **What is it?** — Definition and core question
2. **The Data You Need** — Required fields and structure
3. **How to Do It** — Step-by-step mechanics
4. **What the Output Looks Like** — Example deliverable
5. **Common Pitfalls** — Data quality issues
6. **Scenario Application** — Brief example with counter-metrics
7. **Key Takeaways** — 4-6 bullet summary

### Image Placeholders

Use HTML comments for images not yet created:

```markdown
<!--
IMAGE PLACEHOLDER: [Description]
[Specific details: axes, labels, data]
-->
```

Designer briefs for all images are in `figures/figures.md`.

## Course Philosophy

Key messages to preserve in all materials:

1. **"Your job is never to optimize a metric; it's to improve the experience that the metric measures."**
2. **"Job titles are fragile, responsibilities are durable."**
3. Analytics projects require **adversarial thinking** — counter-metrics, pre-mortems, blocker identification
4. Foundational analyses follow the **customer journey** (Acquisition → Retention → Growth)
5. **Correlation is not causation** — every pre-mortem should consider this trap

---

*Last updated: January 2026*

---
> Source: [earino/designing-analytics-projects](https://github.com/earino/designing-analytics-projects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
