## checks

> How to add or modify a scanner check


Adding a check:

1. Pick the dimension file (`context.ts`, `skills.ts`, `hooks.ts`,
   `sensors.ts`, `ci.ts`, `hygiene.ts`) and the next free ID in that
   family's sequence (e.g. `SNS-06`).
2. A check reads ONLY through `ScanContext` (`has` / `read` / `matching`) —
   never `fs` directly, so tests can fake it.
3. `evidence` must state what was found and where, in one sentence, for both
   pass and fail. `remediation` is one actionable sentence.
4. Keep dimension totals in mind: points must sum to the dimension's
   documented maximum, and the overall total must stay 100.
5. Update the check catalog in `docs/guide/measure-and-improve.md` with a
   `#### <ID> · <title> — <n> pts {#<id-lowercase>}` entry — the docs-sync
   test fails otherwise.
6. Add a unit test in `packages/cli/test/checks.test.ts` (use
   `fakeContext`), and adjust fixtures if the check changes any fixture's
   expected level.

---
> Source: [paladini/harness-score](https://github.com/paladini/harness-score) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
