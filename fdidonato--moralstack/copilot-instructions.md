## documentation-enforcement

> |


# Documentation Enforcement

## When Documentation Must Be Updated

| Change Type                        | Required Documentation Update                     |
|------------------------------------|---------------------------------------------------|
| Public API signature change        | @docs/architecture_spec.md (API contracts section) |
| New module or subpackage           | @docs/modules/ (add or update relevant module doc) |
| Schema change (Pydantic models)    | @docs/architecture_spec.md + module doc            |
| New CLI option or entrypoint       | @README.md (Quickstart) + @INSTALL.md              |
| Risk taxonomy or category change   | @docs/modules/risk_estimator.md                    |
| Constitution schema change         | @docs/constitution.md                              |
| Decision policy change             | @docs/decision_policy.md + @README.md              |
| New dependency                     | @INSTALL.md + pyproject.toml                       |
| Module config loader or env vars   | Module doc in @docs/modules/ + @INSTALL.md + @.env.template |

## Documentation Index

- @README.md — project overview, decision model, quickstart
- @INSTALL.md — installation and environment configuration
- @docs/architecture_spec.md — engineering spec (components, contracts, flows, tests)
- @docs/modules/ — per-module documentation (orchestrator, risk, policy, critic, simulator, hindsight, perspectives, constitution_store, benchmark)
- @docs/constitution.md — constitution design and YAML format
- @docs/decision_policy.md — formal SAFE_COMPLETE / NORMAL_COMPLETE / REFUSE policy
- @docs/DEVELOPMENT.md — development workflow (pytest, ruff, CI)
- @docs/limitations_and_tradeoffs.md — known limitations

## AI Instructions

- After any structural edit, **check** which docs from the table above need updating.
- Do not overwrite existing documentation patterns; extend them consistently.
- If a new public function is added, ensure its docstring follows existing conventions and the relevant module doc in @docs/modules/ is updated.

---
> Source: [fdidonato/moralstack](https://github.com/fdidonato/moralstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
