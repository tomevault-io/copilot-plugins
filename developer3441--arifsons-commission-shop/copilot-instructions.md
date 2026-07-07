## arifsons-commission-shop

> | **Domain context** (actors, lifecycle, worked example) | `docs/blueprint.md` |

# Agent Guide

@CONTEXT.md

## Where things live — read on demand

| Need | Read |
| --- | --- |
| **Domain context** (actors, lifecycle, worked example) | `docs/blueprint.md` |
| **Vocabulary** | `docs/glossary.md` |
| **Rules & why** | `docs/adr/` — index: `docs/adr/README.md`; conventions: `docs/adr/0000-*` |
| **Tech map** (stack, data flow, delivery boundary, where schema/API truth lives) | `docs/architecture.md` |
| **UI standards** (patterns, states, a11y) | `docs/design.md` |
| **What to build** | issue tracker (work SSOT); `docs/prd/current.md` is a single ephemeral scaffold, deleted once its issues are cut |

## Rules of engagement

1. **ADRs are the single source of truth for rules.** Read the relevant ADR before changing logic
   in its area; never silently re-decide a settled one.
2. **Use glossary vocabulary** in code, comments, and docs. New domain terms → `docs/glossary.md`.
3. **Blueprint is narrative, not authority.** If it and an ADR disagree, the **ADR wins**.
4. **Change a decision = supersede, never edit a settled ADR** (clarifications may be edited in
   place). Conventions & supersession protocol: `docs/adr/0000-*`.

*Test-first is the default for domain/business logic — drive it with the `tdd` skill (the red-green method lives there). Skip only for trivial/exploratory code.*

*Finishing an issue: tick its acceptance-criteria boxes (each against the test that proves it), then commit on an issue branch → push → open a PR → merge when CI is green → close. `main` is protected — never push to it directly; CI passing on the PR is the definition of done.*

---
> Source: [developer3441/arifsons-commission-shop](https://github.com/developer3441/arifsons-commission-shop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
