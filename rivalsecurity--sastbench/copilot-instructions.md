## sastbench

> Some directories include a **`TECHNICAL.md`** that documents behavior at the implementation layer (data flow, invariants, file roles)—for example **`dataset_generation/TECHNICAL.md`** and **`evaluation/TECHNICAL.md`**.

# Repository notes for AI assistants

## `TECHNICAL.md` files

Some directories include a **`TECHNICAL.md`** that documents behavior at the implementation layer (data flow, invariants, file roles)—for example **`dataset_generation/TECHNICAL.md`** and **`evaluation/TECHNICAL.md`**.

When you **edit code under a directory that contains `TECHNICAL.md`**, **open that `TECHNICAL.md` and update it if the change affects documented behavior**. Not every edit needs a doc change: purely internal refactors that preserve the same externally visible behavior may need no update. If the change is **below the abstraction level** described there (e.g. a small helper rename), the technical doc often stays unchanged; if it changes **CLI flags, task flow, caches, dataset contracts, or metrics**, the technical doc should stay accurate.

---
> Source: [RivalSecurity/sastbench](https://github.com/RivalSecurity/sastbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
