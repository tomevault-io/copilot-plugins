## fp-guardrails

> Add regression guardrails when fixing rule false positives


# False-positive guardrails

When fixing a **false positive** in any rule:

1. Add a minimal repro to `packages/dbt-doctor/tests/fixtures/dbt-fp-guardrails.ts`
   - One `id` per edge case; list every rule that must stay silent in `rules`.
2. Run `pnpm test` — `dbt-fp-guardrails.test.ts` runs all cases via `it.each`.
3. Optionally validate on a real project: `pnpm fp-scan /path/to/dbt-project --preset enterprise`.

Do not add large project snapshots to the repo; keep fixtures minimal and **anonymized** (no client model names, sources, or comments).

## Coverage gap

~190 rules exist; guardrails currently cover common Jinja/dbt SQL patterns. Style-only rules use dedicated `sql-*-style.test.ts` files. When triaging new FPs, add **anonymized** cases to `DBT_FP_GUARDRAILS` or the relevant style test file.

---
> Source: [northgraindata/dbt-doctor](https://github.com/northgraindata/dbt-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
