## docs-style

> External-docs tone rules for the MkDocs site, README.md, and examples READMEs


# External docs style

When editing the consumer-facing documentation surface, follow the rules in [`spec/internal/docs-style-guide.md`](mdc:spec/internal/docs-style-guide.md). The canonical document is the authoritative source; this file is a loader only.

## Scope

Rules apply to:

- `docs/**/*.md` — **excluding** `spec/internal/**`. Files under `spec/internal/**` are internal analysis and follow different conventions.
- `README.md`.
- `examples/**/README.md`.

## One-line summary of the four rules

- **R1** — Tutorial voice. Describe what the plugin does on its own terms.
- **R2** — No adversarial library comparisons on the published site. *"Pick X when …"*, not *"X is wrong when …"*. Opinion prose belongs under `spec/internal/research/`.
- **R3** — No status / priority / roadmap / phase content. That lives in [`spec/reference/requirements.md`](mdc:spec/reference/requirements.md), which is excluded from the site build.
- **R4** — Avoid rhetorical-contrast vocabulary: *stops short*, *hand-written*, *boilerplate*, *does not do*, *despite the name*, *what … does NOT*.

Read the full style guide before substantive edits to published pages: [`spec/internal/docs-style-guide.md`](mdc:spec/internal/docs-style-guide.md). Enforcement lives in [`.claude/commands/review.md`](mdc:.claude/commands/review.md) Phase 2 as `Req-10`.

---
> Source: [mikezaschka/cds-data](https://github.com/mikezaschka/cds-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
