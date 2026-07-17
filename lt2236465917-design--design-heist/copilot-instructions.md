## design-heist

> This repository contains the `design-heist` Codex skill.

# Repository Guardrails

## Purpose

This repository contains the `design-heist` Codex skill.

The skill is an orchestration workflow for authorized website reconstruction:

- generate a deterministic clone prompt first
- implement only after user confirmation
- use GSAP for complex motion when appropriate
- verify with build checks and screenshots before claiming completion
- debug root causes before visual tweaking

## Boundaries

- Keep the skill free, public, and open source.
- Do not add instructions that encourage unauthorized reuse of third-party brand assets, proprietary copy, paid fonts, paywalled content, or private media.
- Treat third-party sites as internal reference only unless the user owns or has licensed the page and assets.
- The skill may inspect public pages for analysis, but it must not automate creator-center publishing flows.

## Maintenance Rules

- `SKILL.md` is the source of truth for agent behavior.
- `README.md` is the source of truth for human visitors.
- If the workflow changes, update both files in the same change.
- Keep instructions short and operational. Avoid marketing language.
- Validate the skill after edits:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py .
```

---
> Source: [lt2236465917-design/design-heist](https://github.com/lt2236465917-design/design-heist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
