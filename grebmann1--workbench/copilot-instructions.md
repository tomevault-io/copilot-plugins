## agent-planning-roadmap

> Store deferred implementation plans in auto-roadmap and remove them after execution


# Agent Planning Roadmap

- When creating a plan that is not executed immediately, save it under `auto-roadmap/` so it can be resumed later.
- If a plan from `auto-roadmap/` is executed and completed, remove that plan file from `auto-roadmap/` as part of cleanup.
- Do not store transient implementation notes in `auto-roadmap/`; use it for durable future work only.

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
