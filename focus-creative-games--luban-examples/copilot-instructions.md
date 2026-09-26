## luban-examples

> Luban MiniTemplate conventions for AI agents


# Luban MiniTemplate

- Pipeline: Schema → Data → Generate → Runtime.
- Never silently weaken schema to make invalid data pass.
- Register new tables in `Data/__tables__.xlsx` before expecting export.
- Excel sheets require A1 starting with `##`.
- Groups: `c` client, `s` server, `e` editor.
- Prefer `--errorFormat json` when diagnosing failures.
- Prefer `-c schema-json` to inspect compiled schema.

---
> Source: [focus-creative-games/luban_examples](https://github.com/focus-creative-games/luban_examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
