## startup-validation-env-order

> Align startup validation.ts push order with apps/*/.env.example when practical.


# Startup validation order vs env files

Use the **startup-validation-env-order** skill
(`.cursor/skills/startup-validation-env-order/SKILL.md`) for full detail.

## Do

- Add new `results.push(...)` entries in the same **section order** as `apps/<app>/.env.example`.
- Place vars from the **last** env section (e.g. Extensions) at the **end** of
  `validateAllEnvironmentVariables`, with a comment referencing that section.
- Update validation when you add or move keys in `.env.example`.

## Don't

- Insert extension or late-section env validations into an unrelated block (e.g. API
  Configuration) just because the validation `category` label differs from the env header.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
