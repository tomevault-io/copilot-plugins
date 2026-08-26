## ai-native-sdlc

> This repo is a reusable skill and plugin bundle implementing the AI-native SDLC workflow. Follow these conventions when working in it.

# AGENTS.md

This repo is a reusable skill and plugin bundle implementing the AI-native SDLC workflow. Follow these conventions when working in it.

## Source of truth

- The workflow lives in `skills/ai-native-sdlc/SKILL.md`. Read it first, then the references it routes to (`references/playbook.md`, `references/adoption.md`).
- Templates in `skills/ai-native-sdlc/assets/` are copied into target projects — never edited to fit one project.
- `skills/ai-native-sdlc/scripts/init_workflow.py` performs that copy; change the script when the scaffold layout changes.

## Validation

After changing the skill or plugin:

```bash
python3 <skill-creator-path>/scripts/quick_validate.py skills/ai-native-sdlc
python3 <plugin-creator-path>/scripts/validate_plugin.py .
```

Both must pass before finishing. Keep `plugin.json` and `SKILL.md` consistent in name and description; keep the `plugin.json` version in sync with releases.

## Conventions

- Keep `SKILL.md` short; put phase detail in references.
- Keep artifacts (intent/spec/plan templates) generic; org specifics belong in the adopter's own skills and hooks.
- Never add hooks or secrets for a specific adopter into the shared templates.

---
> Source: [bashebr/ai-native-sdlc](https://github.com/bashebr/ai-native-sdlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
