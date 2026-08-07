## skills

> This repository stores public AI-agent skills. Treat every change as something other people may copy into their own agent setup.

# Repository Instructions

This repository stores public AI-agent skills. Treat every change as something other people may copy into their own agent setup.

## Structure

- Store skills under `.agents/skills/<skill-name>/`.
- Every skill must include `SKILL.md`.
- Add `agents/openai.yaml` for Codex UI metadata.
- Use `scripts/`, `references/`, and `assets/` only when they directly support the skill.
- Do not add extra docs inside a skill folder unless they are loaded intentionally by `SKILL.md`.

## Skill Quality

- Keep each skill focused on one recurring job.
- Write the frontmatter `description` as trigger logic, not marketing copy.
- Include likely user phrases in the description when helpful.
- Put the actual operating procedure in the body.
- Prefer concrete steps, failure modes, and validation over broad explanations.
- Keep `SKILL.md` concise; move long examples or background into `references/`.

## Metadata

For `agents/openai.yaml`, use:

```yaml
display_name: Human Name
short_description: Short action phrase
default_prompt: Help me ...
```

Keep `short_description` short and human-facing. Keep `default_prompt` as a realistic first user request.

## Editing Rules

- Preserve existing user-authored skill content unless asked to rewrite it.
- Keep diffs narrow and scoped to the requested skill or repository docs.
- Prefer ASCII in new files.
- Do not create commits unless the user explicitly asks.
- Before finalizing, run `git status --short` and inspect the changed files.

## README Updates

When adding a new skill, update `README.md`:

- Add it to the Available Skills table.
- Link directly to its `SKILL.md`.
- Keep the description short and outcome-focused.

---
> Source: [buhaistrikalo/skills](https://github.com/buhaistrikalo/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
