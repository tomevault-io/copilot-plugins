## hivemind

> Monty is the high-complexity interpreter crate with extensive fixtures and behavioral tests.

# MONTY CRATE KNOWLEDGE

## OVERVIEW
Monty is the high-complexity interpreter crate with extensive fixtures and behavioral tests.

## WHERE TO LOOK
- `src/` — interpreter implementation
- `tests/`, `test_cases/` — executable language and compatibility contracts
- workspace manifests — feature/MSRV/lint constraints

## CONVENTIONS
- Change behavior only with a focused regression test and relevant fixture coverage.
- Keep parser/runtime errors deterministic and user-observable.
- Run the narrow test target before the full executor workspace suite.

## ANTI-PATTERNS
- Do not delete or weaken fixtures to silence failures.
- Do not add network or host filesystem access to interpreter semantics.

---
> Source: [him6794/hivemind](https://github.com/him6794/hivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
