## ai-redteam-recursive-self-improvement

> Before changing this repository, read:

# Agent Rules

## Mandatory Read Order

Before changing this repository, read:

1. `README.md`
2. `CONTRIBUTING.md`
3. `SECURITY.md`
4. `rsi_framework/protocol.py`

## Roles

The framework separates four responsibilities:

- Controller: freezes scope, records the round, and presents decisions.
- Builder: implements only the frozen task.
- Implementation red-team: reviews code, tests, evidence, and reproducibility.
- Research red-team: challenges whether the evidence supports the claimed
  improvement.

No role may bypass unresolved vetoes or change protocol boundaries silently.

## Hard Boundaries

- Preserve failed attempts.
- Keep promotion decisions explicit.
- Treat missing evidence as a blocker.
- Do not rewrite a round outcome after seeing a favorable result.
- Do not merge scope expansion into an implementation-only task.
- Do not treat a builder's claim as independent review.

## Deletion Boundary

Do not delete ledgers, decision records, tests, or failed-attempt artifacts unless
the exact target is named and the deletion is approved.

---
> Source: [lihouwenbin/ai-redteam-recursive-self-improvement](https://github.com/lihouwenbin/ai-redteam-recursive-self-improvement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
