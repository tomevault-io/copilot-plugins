## phase1-freeze

> Phase 1 beta stabilization freeze — no new features until release checklist passes


# Phase 1 Freeze Rule

Until all items in `planning/RELEASE_CHECKLIST.md` pass:

- **No new AI features** (RAG, embeddings, cloud sync, IDE extensions, etc.)
- **No architecture rewrites** — only changes required for C1–C8 blockers
- **No refactors** unless directly required for stability or testability
- **Read `/planning/*`** before starting any implementation work

**Stop condition:** Phase 1 is complete when `planning/RELEASE_CHECKLIST.md` is fully green.

---
> Source: [harsh25jai/ctxstack](https://github.com/harsh25jai/ctxstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
