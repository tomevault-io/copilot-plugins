## to-mars

> These rules apply to every agent in this repository.

# Agent Pack Operating Rules

These rules apply to every agent in this repository.

## Core Behavior

Each agent should produce a decision-ready operating output, not a generic analysis. Every run should answer:

- What changed?
- Why does it matter?
- What should be done next?
- Who needs to act?
- What is uncertain or missing?

## Output Standards

Every output must include:

- Executive summary
- KPI or signal table
- Key insights
- Recommended actions
- Owner / function
- Data gaps
- Confidence level
- Next run checklist

## Data Handling

- Use synthetic, public, or properly authorized data only.
- Do not commit private business data to GitHub.
- Store local data in the generated `workspace/data/` directory.
- Store generated outputs in `workspace/outputs/`.
- Keep assumptions clearly labeled.

## Review Gates

Human review is required before:

- external partner communication
- pricing changes
- customer-facing interventions
- inventory reallocation
- executive escalation
- public use of any company-specific data

## Run Loop

1. Read the agent README and prompt.
2. Check local input files in the workspace.
3. Identify missing data.
4. Generate the agent output.
5. Save the output in `workspace/outputs/<agent-id>/`.
6. Update any follow-up notes in `workspace/NEXT_ACTIONS.md`.

---
> Source: [jjacobsberg/To-Mars](https://github.com/jjacobsberg/To-Mars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
