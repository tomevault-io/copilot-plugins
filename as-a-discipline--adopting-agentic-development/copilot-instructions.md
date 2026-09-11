## adopting-agentic-development

> * Primary engineering instructions for this repository are in the root

# Copilot Instructions

* Primary engineering instructions for this repository are in the root
  [`AGENTS.md`](../AGENTS.md).
* Nested `AGENTS.md` files (e.g. `api/AGENTS.md`, `service/AGENTS.md`) contain
  component-specific instructions — read the nearest one before changing that
  component.
* Reusable workflows are in [`.agents/skills/`](../.agents/skills/); load a skill
  when its trigger matches the current task.
* Perform project actions through documented `task` targets, not invented shell
  commands.
* Inspect the repository's actual current state rather than assuming prior
  conventions or a previous session's work is correct.

---
> Source: [as-a-discipline/adopting-agentic-development](https://github.com/as-a-discipline/adopting-agentic-development) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
