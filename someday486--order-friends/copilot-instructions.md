## order-friends

> Read these files before making non-trivial changes:

# Order Friends - Copilot Instructions

Read these files before making non-trivial changes:

1. `README.md`
2. `docs/ENGINEERING_WORKFLOW.md`
3. `docs/DOCUMENT_REGISTRY.md`

## Required Workflow

- Follow the document lifecycle rules from `docs/ENGINEERING_WORKFLOW.md`
- Do not add temporary summary/task/fixes markdown files at the repository root
- Put new docs in the correct `docs/` subfolder instead of the root
- Add a patch note in `docs/patch-notes/` for user-facing or operator-facing changes

## High-Risk Areas

If a change touches any of these, describe impact and validation in the PR:

- auth or permissions
- payments
- order status logic
- database migrations or triggers
- large data updates

## Documentation Discipline

- Keep `docs/README.md` as the entry point
- Keep `docs/DOCUMENT_REGISTRY.md` updated when document status changes
- Move obsolete or one-off notes to `docs/archive/`

---
> Source: [someday486/order_friends](https://github.com/someday486/order_friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
