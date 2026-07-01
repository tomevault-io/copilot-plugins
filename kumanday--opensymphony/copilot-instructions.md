## opensymphony

> This target repository exists to validate the local OpenSymphony MVP on a trusted machine.

# Example Target Repo

This target repository exists to validate the local OpenSymphony MVP on a trusted machine.

- `WORKFLOW.md`, this `AGENTS.md`, and optional `.agents/skills/` entries are repo-owned policy.
- `.opensymphony/` and `.opensymphony.after_create.json` are local OpenSymphony artifacts for audit and recovery.
- Inspect generated OpenSymphony artifacts when needed, but do not rewrite repo-owned policy files to steer the run.
- Keep commands deterministic.
- Do not write outside the target workspace.
- Prefer small edits that are easy to verify during smoke and live validation.

---
> Source: [kumanday/OpenSymphony](https://github.com/kumanday/OpenSymphony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
