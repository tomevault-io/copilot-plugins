## stackscan

> You have access to a stackscan skill plugin in this repository. When the user asks to analyze their operations, follow the pipeline defined in `skills/continue/SKILL.md`. For new projects, use `skills/new/SKILL.md`.

# StackScan — Operational Investment Decision Engine

You have access to a stackscan skill plugin in this repository. When the user asks to analyze their operations, follow the pipeline defined in `skills/continue/SKILL.md`. For new projects, use `skills/new/SKILL.md`.

## Available Skills
- **New project:** Read and follow `skills/new/SKILL.md` — intake interview + full 15-step analysis
- **Continue analysis:** Read and follow `skills/continue/SKILL.md` — resume 15-step pipeline on existing project
- **Dashboard:** Read and follow `skills/dashboard/SKILL.md` — generate HTML dashboard

## How to Use
1. Read the relevant SKILL.md file for the full instructions
2. Read prompt templates from `skills/continue/prompts/` as needed
3. Read reference files from `references/` for shared protocols
4. Read templates from `templates/` for intake forms
5. Write all outputs to `~/.stackscan/projects/{project}/` in the user's home directory

## Key Rules
- Frame everything as investment decisions, not process improvements
- Every number must show its derivation: `result (= formula)`
- Mine data before asking the user (Data Resolution Protocol)
- Steps 3, 6, 7 require user confirmation before proceeding
- ROI.md is the centerpiece output — everything feeds into or follows from it

## The 15-Step Pipeline
1. Create plan  2. Research tools  3. Analyze inefficiencies  4. Evaluate tool fit
5. Discover tools  6. Design process  7. Calculate ROI  8. Assess risks
9. Generate diff  10. Adoption plan  11. Role guides  12. Integration checklist
13. Consistency check  14. Action plan  15. Executive summary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyperop-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
