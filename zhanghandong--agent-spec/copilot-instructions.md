## agent-spec

> 1. Read the contract: `agent-spec contract specs/<task>.spec`

# agent-spec Integration for Cursor

## Workflow

1. Read the contract: `agent-spec contract specs/<task>.spec`
2. Implement against the contract's Intent, Decisions, and Boundaries
3. Verify: `agent-spec lifecycle specs/<task>.spec --code . --format json`
4. Guard: `agent-spec guard --spec-dir specs --code .`
5. Review: `agent-spec explain specs/<task>.spec --code . --format markdown`

## Key Commands

- `agent-spec contract <spec>` — render the Task Contract for planning
- `agent-spec lifecycle <spec> --code <dir>` — full lint + verify pipeline
- `agent-spec guard --spec-dir specs --code .` — repo-wide pre-commit check
- `agent-spec explain <spec>` — human-readable contract review summary
- `agent-spec stamp <spec> --dry-run` — preview git trailers

## Retry Protocol

When `lifecycle` fails:
1. Parse JSON output, find each scenario's `verdict` and `evidence`
2. For `fail`: read evidence, fix code
3. For `skip`: check `Test:` selector matches a real test name
4. **Fix code based on evidence. Do NOT modify the spec file.**
5. Re-run `lifecycle`
6. After 3 consecutive failures on the same scenario, escalate to the human

## Conventions

- Task specs live in `specs/`
- Verdicts: pass, fail, skip, uncertain — all four are distinct
- **skip ≠ pass**: skipped scenarios block the pipeline
- Exception scenarios should be >= happy path scenarios

---
> Source: [ZhangHanDong/agent-spec](https://github.com/ZhangHanDong/agent-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
