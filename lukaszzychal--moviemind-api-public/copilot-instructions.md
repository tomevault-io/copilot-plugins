## 010-scope-cost

> - Work only on files explicitly provided by the user or explicitly requested (max 2 files per iteration).


# Scope and cost control

- Work only on files explicitly provided by the user or explicitly requested (max 2 files per iteration).
- Never request or attempt a full repository analysis.
- Prefer: short plan -> small patch -> run tests -> next step.
- Never include these paths in context: vendor/, node_modules/, storage/, bootstrap/cache/, var/cache/, logs/, coverage/.
- If you need to locate code: request a targeted search result (file + line + short snippet), not full file dumps.

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
