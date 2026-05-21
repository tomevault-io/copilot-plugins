## kowalski-planning-and-boundaries

> Kowalski task plans, AGENTS-first workflow, core vs CLI boundaries


# This repository (Kowalski)

- Follow **root `AGENTS.md`** for philosophy, **SOLID** (see `tools/solid_principles_quick_reference.md`), **Rule 0** (read the **component `AGENTS.md`** before editing that crate), **Rule 7** (docs with behavior/API changes), and error/logging protocols.
- For planned work, sany examples / ingest / federation: open **`task_plan.md`**, respect numbered phases and “pending vs complete”; when a phase completes or scope shifts, update **`progress.md`** and non-obvious decisions in **`findings.md`**.
- **Architecture**: ingest and URL bundling live in **`kowalski-core`** (`source_bundle`, `tools::internal::*`). **`kowalski-cli`** and **`ui/`** stay **thin**—no horde-specific fetch or domain orchestration in CLI beyond invoking core / APIs.

---
> Source: [yarenty/kowalski](https://github.com/yarenty/kowalski) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
