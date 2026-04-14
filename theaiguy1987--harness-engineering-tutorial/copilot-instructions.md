## harness-engineering-tutorial

> > Your map for this project. Keep it open while you work.

# GEMINI.md

> Your map for this project. Keep it open while you work.

---

## What this project is

The goal is simple: use Gemini CLI to **read a real financial report and turn it into a presentation**.

The report is Volvo Group's Q4 and Full Year 2025 earnings (`volvo-group-q4-2025-eng.pdf`).
Your job — guided by the agent — is to extract the key findings and produce a slide-ready executive brief suitable for a non-specialist audience.

This repo is also a live example of **Harness Engineering**: the GEMINI.md you are reading right now, combined with the three skills below, is the *harness* — a set of encoded human instructions that give the agent direction, taste, and guardrails. The agent supplies the horsepower; the harness tells it where to go.

---

## The pipeline

Three skills are installed in `.gemini/skills/`. They run in sequence:

| Step | Skill | What it does | Output |
|------|-------|--------------|--------|
| 1 | `pdf-parser` | Extracts text and tables from the PDF | `parsed-report.md` |
| 2 | `financial-analysis` | Analyzes the parsed content; applies your perspective | `financial-analysis.md` |
| 3 | `executive-brief` | Converts the analysis into a PowerPoint deck | `executive-brief.pptx` |

Run the full pipeline with one prompt:

> *Parse the Volvo Q4 2025 PDF, run the financial analysis, then create the executive brief.*

---

## Step 0 — Environment setup (do this first)

Before running any skill, make sure the Python virtual environment `demo-venv` is active.

```bash
# First time — create and install
python -m venv demo-venv
demo-venv\Scripts\activate        # Windows
pip install -r requirements.txt

# Returning — just activate
demo-venv\Scripts\activate        # Windows
```

> **Agent:** Check for `demo-venv` on startup. Create and install if missing; activate if it exists. Do not run any skill until the environment is confirmed active.

---

## Output rules

- All figures must come directly from the PDF — never estimate or invent
- Always show year-on-year context (2025 vs 2024) when quoting a number
- Write for an intelligent non-specialist: plain language, no jargon
- Bullet points over paragraphs; every section should fit on one slide
- All amounts in SEK unless stated otherwise

---

## Personalise the analysis

Open `.gemini/skills/financial-analysis/SKILL.md` and fill in the **About me** section.
Re-run the `financial-analysis` skill and see how the output shifts to reflect your perspective.
That loop — run → review → refine → re-run — is the core of harness engineering.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theaiguy1987)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/theaiguy1987)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
