## ai-asset-pricing

> This file is auto-loaded by Gemini CLI. It imports the shared routing layer

# Gemini CLI Adapter

This file is auto-loaded by Gemini CLI. It imports the shared routing layer
and adds Gemini-specific notes.

@AGENTS.md

## Gemini-Specific Notes

- Onboarding: ask Gemini CLI to onboard or set up the repo in chat. Under the
  hood it should use the same cold-start flow as Claude `/onboard` and Codex:
  `tools/onboard.ps1` or `tools/onboard.sh` to get Python 3.11+, then
  `tools/onboard_driver.py`, then `tools/bootstrap.py audit`, the emitted
  bootstrap plan, and `tools/bootstrap.py apply`. WRDS is optional and should
  only be configured if the user has an account and wants it set up now.
- Skills: read `.claude/skills/*/SKILL.md` files directly. They are plain
  markdown instructions, not Claude-binary artifacts. Gemini can follow them
  as-is.
- Figures: for charts, FT-style plots, house-style publication plots,
  validation galleries, or Word proof packs, read `docs/ai/figures.md` and use
  `fintools.figures` before writing custom plotting code. Use `style="fins"`
  for the house style and `style="ft"` for FT-style output; legacy
  `.claude/skills/publication-figures/` helper assets remain available when a
  standalone helper workflow is explicitly requested.
- Domain experts: read `.claude/agents/*.md` for deep WRDS, CRSP,
  OptionMetrics, TAQ, bonds, and Fama-French domain context.
- Hooks: `.claude/hooks/` are Claude Code automation (LaTeX rebuild,
  pre-commit preflight). Gemini CLI users should run
  `tools/release_preflight.py --strict` manually before commits.
- Strict preflight auto-cleans repo temp artifacts when possible and tolerates
  gitignored repo-root local artifacts such as `.venv/`,
  `.claude/settings.local.json`, `.tmp-pytest-current/`, `.tmp-uv-cache/`, and
  `.Rhistory`; `LOCAL_ENV.md` and `CLAUDE.local.md` still block release.
- MCP: Perplexity MCP is configured in `.claude/settings.json`. Gemini users
  should configure equivalent MCP servers in `.gemini/settings.json` if needed.
- The canonical local state lives outside the repo. See `docs/ai/onboarding.md`.

---
> Source: [Alexander-M-Dickerson/ai-asset-pricing](https://github.com/Alexander-M-Dickerson/ai-asset-pricing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
