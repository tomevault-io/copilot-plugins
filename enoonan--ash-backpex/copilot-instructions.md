## ash-backpex

> Ash Backpex is an Elixir library that integrates Ash resources with Backpex admin LiveViews using a Spark DSL and compile-time code generation.

# AGENTS.md

## Project

Ash Backpex is an Elixir library that integrates Ash resources with Backpex admin LiveViews using a Spark DSL and compile-time code generation.

## Commands

- Install deps: `mix deps.get`
- Run tests: `mix test`
- Run one test: `mix test test/ash_backpex/adapter_test.exs`
- Format: `mix format`
- CI checks: `mix ci`

## Notes

- Read `usage-rules.md` before changing library behavior or examples.
- Core code lives under `lib/ash_backpex/`.
- The main entry point is `AshBackpex.LiveResource`; the custom adapter is `AshBackpex.Adapter`.
- Tests use in-memory SQLite fixtures from `test/support/`.
- There is a demo Phoenix app under `demo/`.
- Preserve unrelated local changes in the working tree.

---
> Source: [enoonan/ash_backpex](https://github.com/enoonan/ash_backpex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
