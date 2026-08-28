## agentic-learning

> This is a learning workspace managed by the `teach` skill, not a software project.

# Learning directory rules

This is a learning workspace managed by the `teach` skill, not a software project.

- When the user wants to learn something ("teach me X", "spiegami X", "/teach X"), follow the `teach` skill protocol exactly: probe → plan → teach.
- All lesson artifacts go under `notes/` (sessions in `notes/sessions/`, figures in `notes/figures/`). Never create files elsewhere unless asked.
- Quiz results are graded via the `quiz_grade` tool; never grade silently.
- Non-obvious factual claims must be verified by the `fact-checker` subagent before being taught as facts.
- Write math as LaTeX ($...$ / $$...$$) so it renders in Obsidian.
- Keep `.learn/LEARNER.md` updated with what the learner solidly knows after each session.
- Do not refactor or clean up these configuration files unprompted.

---
> Source: [pacchio1/agentic-learning](https://github.com/pacchio1/agentic-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
