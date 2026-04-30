## bsec-2026

> - This is a hackathon project.

# AGENTS.md

## Project mode
- This is a hackathon project.
- Priority order: `working demo > simplicity > speed of debugging > polish`.
- Keep code and architecture easy to understand and easy to modify quickly.
- Security hardening is **not** a priority unless it blocks the demo.

## Goal
Build a functional multi-agent editorial system that generates content for:
- YouTube (video script)
- Instagram (post)
- TikTok (reel)

The system should use AI where it helps, and avoid unnecessary AI calls.

## Required agents
- `Author`: proposes content ideas / drafts.
- `Editor`: adapts content per platform.
- `QA`: reviews output quality and makes approve/revise decision.

## Required use cases
Implement exactly these 2 flows:
1. Generate topic options to choose from.
2. Accept a short input brief and generate platform outputs.

## Minimum input format
Support this structure (or equivalent fields):
- `tema` (topic): `"AI automatizace marketingu"`
- `cil` (goal): `"edukovat"`
- `cilova_skupina` (target audience): `"zakladatelé startupů..."`
- `ton` (tone): `"profesionální, ale srozumitelný"`
- `hlavni_myslenka` (main idea): `"AI není náhrada..."`
- `cta`: `"Přihlaste se k newsletteru"`

## Required outputs
Each run must return all 3:
- YouTube video script
- Instagram post
- TikTok reel

## History + data requirements
- Use published-content history to influence new outputs.
- Use provided dataset (`important_files/Styl Čendy dataset.xlsx`) as example creator history.
- Keep solution creator-agnostic (not hardcoded only for Čenda).

## Web trends
- Include web search/trend enrichment in workflow (Vertex AI search recommended in assignment).
- Use this Vertex service account file for setup: `important_files/red-splice-488519-s6-60554b78a80a.json`

## QA requirements
QA agent must produce structured review with:
- pass/fail (approve/revise)
- short reason
- recommended edits
- checks for platform format/length/structure
- relevance/originality vs history + trends

## Performance and cost
- End-to-end flow should complete in a few minutes.
- Minimize token usage and unnecessary model calls.
- Load only needed context for each step.

## Frontend requirements
Frontend must:
- support both required use cases
- show full workflow status by step
- display final outputs clearly

## Demo/presentation constraints
- Include automatic email send to judges before presentation with final output.
- Demo is a live run from prompt to final result.
- Be ready to explain architecture.
- Be ready to explain the agent list.
- Be ready to explain token/cost minimization strategy.

## Scoring focus (from brief)
- Architecture quality: `20%`
- Use of history + web data + optimization: `20%`
- Output quality: `20%`
- Creativity (including optional bonus agents/tools): `20%`
- UX and presentation quality: `20%`

## Engineering style for this repo
- Prefer clear modules and explicit data flow over abstractions.
- Keep functions small and inspectable.
- Add logs around each workflow step for quick debugging.
- Write only essential tests/smoke checks needed for demo confidence.
- Avoid premature refactors.

---
> Source: [olexamatej/bsec-2026](https://github.com/olexamatej/bsec-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
