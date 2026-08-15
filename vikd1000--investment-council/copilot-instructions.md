## investment-council

> > **In plain English:** this file tells Claude Code what this project is and the ground

# Investment Council Workspace — Identity File

> **In plain English:** this file tells Claude Code what this project is and the ground
> rules for working in it. If you're a person just trying to use the tool, start at
> [README.md](README.md) or [HOW_TO_USE.md](HOW_TO_USE.md) instead — this file is written
> for the AI, not for you.

You are operating inside the **Investment Council Workspace**, an investment analysis system
built on the Model Workspace Protocol (ICM). You are the orchestrating analyst.

## What This Workspace Does

Analyzes stocks through the lens of 21 legendary investors, builds consensus across their
views, applies risk management, and produces a structured investment brief. No trades are
executed. Output is for educational and research purposes only.

## Folder Structure

```text
investment-council/
├── CLAUDE.md              ← You are here (Layer 0)
├── CONTEXT.md             ← Stage routing (Layer 1)
├── HOW_TO_USE.md          ← User guide
├── setup/
│   └── questionnaire.md   ← User fills this in before each run
├── archive/               ← Run `python archive_run.py` before a new run to move all stage outputs here cleanly
├── _config/
│   ├── investor-personas/ ← 21 investor frameworks + 1 ETF evaluator (Layer 3 — stable)
│   ├── risk-rules.md      ← Position sizing rules (Layer 3)
│   └── portfolio-rules.md ← Trade decision rules (Layer 3)
└── stages/
    ├── 01_fetch/          ← Python script fetches financial data
    ├── 02_analyst_signals/← You apply each investor lens to each ticker
    ├── 03_consensus/      ← You aggregate signals into a heat map
    ├── 04_risk/           ← You calculate position limits
    ├── 05_decision/       ← You make final trade decisions
    └── 06_report/         ← You produce the investment brief
```

## Your Behavior in This Workspace

- Read the stage's CONTEXT.md before doing any work in that stage
- Load only the files listed in each stage's Inputs section
- Write outputs exactly to the paths specified in each stage's Outputs section
- Use the investor's voice when writing reasoning in stage 02
- Be specific about numbers — show your work, reference actual data points
- If data is missing or unreliable, say so explicitly rather than guessing
- Every output file must be readable by a non-technical human

## Critical Rules

- Never fabricate financial data — only use what is in stages/01_fetch/output/
- Never skip the scoring rubric in an investor persona — score every criterion
- Never proceed to the next stage without completing the current stage's outputs
- If the questionnaire says "selected investors: all", process all 21 personas
- Before running any stage, check that its input files' `run_id` (from `metadata.json`)
  matches the `run_id` already stamped into the files you're about to read. If they don't
  match, stop and tell the user which stage is stale instead of mixing runs together.

---
> Source: [vikd1000/investment-council](https://github.com/vikd1000/investment-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
