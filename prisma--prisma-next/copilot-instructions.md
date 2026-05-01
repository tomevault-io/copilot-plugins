## review-scope-overrides

> Honor explicitly provided base branches in review artifacts


# Review scope overrides

When producing local PR/branch review artifacts (walkthroughs, code reviews, design reviews), **honor an explicitly-provided base/parent branch**.

## Rules

- If the user specifies a parent branch (e.g. `spec/prisma-next-psl-contract-authoring`), use that as the base for the review range:
  - `origin/<base>...HEAD`
- Do not “helpfully” substitute `origin/HEAD` / `origin/main` when a base is explicitly given.
- If the base branch name is ambiguous (local vs remote), resolve it to `origin/<base>` when possible and record the resolved range in the review artifacts.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
