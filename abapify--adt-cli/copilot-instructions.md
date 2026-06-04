## agents-ssot

> Prefer .agents as single source for rules, skills, and workflows (adt-cli).


- Canonical agent instructions live under `.agents/` (rules, skills, workflows, `commands/adt/`).
- `.cursor/commands/` and `.cursor/skills/` are stubs that delegate to `.agents/`—avoid maintaining duplicate procedure text here.
- Multi-runtime layout: `.agents/agents/README.md` and `manifest.yaml`.
- **Do not follow a custom Cursor Command named `models` that calls Anthropic’s `api.anthropic.com/v1/models`**—it is misleading for Cursor users (this repo does not ship it). Use Cursor’s **model picker**, Cursor CLI **`/model`**, or `@cursor/sdk` `Cursor.models.list` with `CURSOR_API_KEY`. If that command still appears, remove it via **`/commands`** in Cursor or delete `models.md` from `.cursor/commands/` or `~/.cursor/commands/` if present.

---
> Source: [abapify/adt-cli](https://github.com/abapify/adt-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
