## serenity-invest-skill

> This repository is a single Agent Skill. Keep the repository root installable as a skill folder.

# Agent Instructions for serenity-invest-skill

This repository is a single Agent Skill. Keep the repository root installable as a skill folder.

## Hard requirements

- `SKILL.md` must remain at the repository root.
- The parent directory name must remain `serenity-invest-skill` unless the `name` field is changed to match.
- `SKILL.md` frontmatter must include `name` and `description`.
- Keep `name` lowercase, hyphen-separated, and under 64 characters.
- Keep `description` under 1024 characters and clear enough for implicit activation.
- Do not add dependencies that are required for skill activation.
- Keep scripts optional and self-contained.

## Financial research guardrails

- Never turn this skill into a buy/sell recommender.
- Always require current sources for market prices, holdings, filings, earnings, policy, or company claims.
- Require evidence before mapping a company to a customer or bottleneck node.
- Keep disconfirmation signals in every investment output.
- Treat small-cap/illiquid stocks as high risk unless evidence and liquidity are explicitly verified.

## Style

- Write in English.
- Prefer concise, operational instructions in `SKILL.md`.
- Put detailed background, examples, and templates in `references/` and `examples/`.

---
> Source: [lazyhuman-ai/serenity-invest-skill](https://github.com/lazyhuman-ai/serenity-invest-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
