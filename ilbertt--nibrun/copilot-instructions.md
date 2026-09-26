## nibrun

> Scripts the repo runs outside the apps — CI steps, deploys, maintenance. Add

# @repo/internal-scripts

Scripts the repo runs outside the apps — CI steps, deploys, maintenance. Add
one here instead of writing a shell script.

- One script file in `src/` = one entry in `package.json`'s `scripts`, named after
  the file.
- Anything shared between scripts lives in `src/shared/`.
- Invoke from anywhere in the repo:
  `bun run --filter @repo/internal-scripts <script>`.

---
> Source: [ilbertt/nibrun](https://github.com/ilbertt/nibrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
