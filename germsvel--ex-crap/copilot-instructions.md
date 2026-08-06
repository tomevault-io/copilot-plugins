## ex-crap

> - This is an Elixir/Mix library and Mix task for CRAP scoring.

# AGENTS.md

- This is an Elixir/Mix library and Mix task for CRAP scoring.
- Use `mix precommit` as the main local quality gate before claiming work is ready.
- For focused changes, run the relevant test file with `mix test path/to_test.exs` first.
- Do not edit generated artifacts in `doc/`, `cover/`, or `tmp/` unless explicitly asked.
- Boundary snapshots are checked with `mix boundary.spec.check`.
- Never manually edit `priv/boundary_spec.txt`; boundary spec updates require
human approval through `mix boundary.spec.check` and `mix boundary.spec.accept`

---
> Source: [germsvel/ex_crap](https://github.com/germsvel/ex_crap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
