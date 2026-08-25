## alfredotheclerk

> This file contains repository-wide instructions for development agents working on

# AGENTS.md

This file contains repository-wide instructions for development agents working on
AlfredoTheClerk.

## Read before changing code

1. Read `ROADMAP.md` for durable product direction and scope.
2. Read `SPEC.md` for the current implementation sequence and acceptance gates.
3. Inspect the current code and tests before assuming a documented feature is
   missing or complete.

Do not use `ROADMAP.md` as a task tracker. Do not put durable product strategy in
`SPEC.md`.

## Repository workflow

- Never commit or push directly to `main`.
- Work on a dedicated branch and open or update a pull request.
- Keep changes narrowly scoped to the requested work.
- Do not introduce speculative abstractions before a second real use case makes
  the shared boundary necessary.
- Preserve unrelated user changes.
- Before claiming completion, inspect the diff and verify the relevant tests.
- Any GitHub comment or review posted on the user's behalf must state that it was
  prepared with AI assistance or is an AI-powered review.

## Product and architecture invariants

- The canonical domestic VAT shell is the business-truth object.
- FA(3) XML is a downstream compliance artifact, not the source of truth.
- Extraction should produce the shell, evidence, diagnostics, validation, and
  extracted summary before agentic repair.
- The agent may choose evidence-backed candidates but must not invent unsupported
  values.
- The agent must not bypass the shell or emit XML directly as the primary repair
  output.
- Agent and human repairs must pass the same deterministic correctness pipeline.
- Do not mark an invoice ready based only on field-level shell validity.
- Monetary summaries must be computed deterministically from canonical line
  items and reconciled with extracted source totals.
- Local XSD validity and remote KSeF acceptance are separate outcomes.
- Unsupported or ambiguous cases must stop for review rather than silently
  degrade.

## Current scope

Unless `ROADMAP.md` explicitly changes the product direction, keep implementation
work focused on:

- ordinary Polish domestic VAT invoices
- native, text-based PDF output
- deterministic extraction with evidence and diagnostics
- evidence-constrained repair and manual escalation
- FA(3) generation and KSeF submission

Do not expand into scanned PDFs, OCR-first extraction, photos, non-domestic
invoices, correction invoices, or advance invoices as part of unrelated work.

## Benchmark and fixture contracts

- Benchmark cases must remain reviewable on disk and loadable without
  regenerating them from seeds.
- Score only fields that are both present in canonical truth and visible in the
  relevant template.
- `payment_form` is scored only when it is present in truth and visible in the
  template.
- Keep `generated_at` fixed inside benchmark cases so target XML is stable.
- Line-item row identity remains position-based until an explicit replacement
  policy is introduced.
- Use scoped validation for the extraction surface being tested; do not turn
  intentionally non-rendered fields into failures.
- Preserve per-template visibility manifests and persisted comparison policies.
- Keep synthetic fixtures after real invoices are added. Real cases supplement
  the deterministic regression corpus rather than replace it.

### Fixture serialization

- Money values: JSON strings in fixed-point form using the repository's
  `round_money` result.
- `quantity`: plain decimal JSON string, no scientific notation, preserve the
  exact value, maximum 6 fractional digits.
- `unit_price_net`: plain decimal JSON string, no scientific notation, preserve
  the exact value, maximum 8 fractional digits.
- `vat_rate`: canonical business-value string such as `"23"` or `"5"`.
- Dates: ISO `YYYY-MM-DD` strings.
- Enums: serialize `.value`.
- Nested dataclasses: nested JSON objects.
- Omit optional fields when their value is `None`.
- Increment the benchmark-case schema version when these rules change.

## Implementation quality

- Prefer the simplest change that satisfies the current acceptance criteria.
- Reuse existing validation, summary, mapping, XML, and XSD code rather than
  duplicating a second correctness path.
- Return structured outcomes for expected business failures; do not hide them in
  generic exceptions.
- Preserve evidence and diagnostics across repair and review boundaries.
- Add regression tests for every fixed failure mode.
- When a real invoice reveals a failure, classify the failing stage before adding
  source-specific logic.
- Do not weaken tests to make a change pass.

## Verification

Run the narrowest relevant tests first, then before completion run:

```bash
uv run ruff check src tests
uv run pytest
uv run python -m compileall src tests
```

For documentation-only changes, CI is still the final repository-level check.

## Documentation ownership

- `ROADMAP.md`: durable product vision, scope, architecture principles, rollout,
  and product success criteria.
- `SPEC.md`: current features, implementation order, statuses, requirements, and
  acceptance criteria.
- `AGENTS.md`: development process, architecture invariants, benchmark contracts,
  Git rules, and verification instructions.

When a task is completed, update its status in `SPEC.md`. Change `ROADMAP.md` only
when the global product direction changes. Do not delete an active contract from
one document unless it is intentionally removed or preserved in the correct
replacement document.

---
> Source: [komaksym/AlfredoTheClerk](https://github.com/komaksym/AlfredoTheClerk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
