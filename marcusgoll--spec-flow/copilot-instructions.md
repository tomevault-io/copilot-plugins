## spec-flow

> - Shared canon lives in `.spec-flow/` (repo map, domain guides, state schemas) and `epics/<slug>/state.yaml`.

# Spec-Flow + Cursor ground rules
- Shared canon lives in `.spec-flow/` (repo map, domain guides, state schemas) and `epics/<slug>/state.yaml`.
- `.claude/` is read-only for Cursor; use it only as reference for how phases should behave.
- Cursor-specific prompts/adapters belong in `.cursor/`.
- Run only one epic phase per session unless explicitly told to chain spec → clarify → plan.
- Always update the relevant `state.yaml` and stop at manual gates before editing subsequent phases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcusgoll) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
