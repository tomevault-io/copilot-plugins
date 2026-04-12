## idse-developer-agent

> IDSE Companion bootstrap rule


# IDSE Companion Bootstrap
- Ensure `.idse_active_session.json` is present if working with Agency-managed sessions; otherwise fall back to simple paths.
- Use `python .cursor/tasks/governance.py` to view session and resolved artifact paths.
- Avoid writing governance artifacts into protected paths (`idse-governance/`, `.idse-layer`, `.cursor/config/idse-governance.json`).
- Run validators before advancing stages: `python integrations/claude-skill/scripts/validate_artifacts.py . --json`.
- Store feedback under `feedback/projects/<project>/sessions/<session_id>/` using `utils/feedback_writer.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tjpilant)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tjpilant)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
