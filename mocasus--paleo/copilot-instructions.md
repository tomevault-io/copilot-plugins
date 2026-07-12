## paleo

> Token-saving skill collection. Skills make LLM usage cheaper: compress output, trim context, skip filler.

# paleo — repo guidance (Gemini CLI)

Token-saving skill collection. Skills make LLM usage cheaper: compress output, trim context, skip filler.

- Skills live in `skills/<name>/SKILL.md` with `name` + `description` frontmatter.
- Installable as a Gemini extension via `gemini-extension.json`.
- Register new skills in `gemini-extension.json` (skills array) + `.claude-plugin/plugin.json`.
- Keep `README.md` version badge (badge row + footer) + plugin `version` in sync.
- Never commit `.env` / `*.session` (git-ignored).

## Token budget (hard rule)
Skills load into the LLM context, so keep them token-minimal: terse prose (drop filler, short clauses), exact code only. Smaller SKILL.md = cheaper + faster agents.

---
> Source: [mocasus/paleo](https://github.com/mocasus/paleo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
