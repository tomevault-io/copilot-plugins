## project-agent-constraints

> Core agent constraints for staged remediation work in this repository


# Project Agent Constraints

- Chat with the user in Simplified Chinese.
- Keep repository files, comments, documentation, task notes, and generated artifacts in English.
- Use `AGENTS.md` and `CLAUDE.md` as the project-level agent entry points.
- For remediation work, follow `docs/audits/full-project-review-2026-04-26/09-task-cards.md`.
- Execute one task card at a time and respect its files in scope, files out of scope, validation commands, and stop conditions.
- Use `docs/audits/full-project-review-2026-04-26/11-execution-sequence.md` for task order.
- Use `docs/audits/full-project-review-2026-04-26/10-implementation-review-checklist.md` before accepting implementation output.
- Use the project skills in `.cursor/skills/` when a task matches the skill description.
- Use `validation-test-runner` for scoped validation after implementation.
- Use `git-change-management` before preparing commits or handoff summaries.
- Use `local-deployment-operations` for local Docker Compose and smoke testing work.
- Do not add dependencies, change migrations, broaden authentication behavior, or expand across workstreams unless the task card explicitly permits it.
- Do not revert unknown user changes or run destructive Git commands.
- After each completed audit task card, create one `git commit` per `AGENTS.md` (English message, task ID in subject when applicable; no push unless asked).

---
> Source: [bigballgis/Document-Generation-System](https://github.com/bigballgis/Document-Generation-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
