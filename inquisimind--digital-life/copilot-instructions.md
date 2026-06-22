## digital-life

> Guidance for Gemini CLI working in Digital Life.

# GEMINI.md

Guidance for Gemini CLI working in Digital Life.

## Required Route

1. Follow [docs/development/development-workflow.md](docs/development/development-workflow.md).
2. Apply [docs/development/spec-kit-policy.md](docs/development/spec-kit-policy.md).
3. Declare `Spec Kit Mode: full | lightweight | none` and a reason before implementation.
4. Use [docs/ai/context-loading-guide.md](docs/ai/context-loading-guide.md) to load only relevant context.
5. Read [docs/architecture/current-system.md](docs/architecture/current-system.md) for current ownership and boundaries.
6. For Python work, read [docs/development/python-coding-standards.md](docs/development/python-coding-standards.md) and [docs/development/python-testing-and-review.md](docs/development/python-testing-and-review.md).

Use installed `/speckit.*` commands for active `full` or `lightweight` work. Load
only the active `specs/{feature}/`. Preserve unrelated work and keep durable
development rules in `docs/development/`.

Use `/python.development` for Python implementation or refactoring and
`/python.review` for findings-first Python review.

<!-- SPECKIT START -->
For active `full` or `lightweight` Spec Kit work, read the current feature plan
after applying the Spec Kit gate. Do not load feature plans for ordinary tasks.
<!-- SPECKIT END -->

---
> Source: [InquisiMind/digital-life](https://github.com/InquisiMind/digital-life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
