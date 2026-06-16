## agent-validation-discipline

> Validate changes with the smallest relevant repo command and report coverage honestly


# Agent Validation Discipline

- Read nearby scripts, config, and existing workflows before claiming validation coverage.
- Run the smallest relevant verification for the files you changed instead of defaulting to `npm run validate`.
- Use repo-real commands when they match the touched area, such as `npm run lint`, `npm run build:shared`, `npm run build:server`, `npm --prefix apps/ui run typecheck`, and `npm --prefix apps/docs run typecheck`.
- Treat bundling, linting, and typechecking as different signals; do not describe one as if it proved the others.
- If a touched area has limited or non-strict type coverage, explicitly say what you ran and what remains unverified.

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
