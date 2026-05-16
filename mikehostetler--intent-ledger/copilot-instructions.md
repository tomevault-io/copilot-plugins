## intent-ledger

> This repository contains the `intent_ledger` Elixir package.

# Agent Notes

This repository contains the `intent_ledger` Elixir package.

## Commands

- `mix test` - run the focused test suite.
- `mix quality` - run the package quality gate.
- `mix docs` - generate HexDocs locally.

## Conventions

- Public modules live under `IntentLedger`.
- Core structs should expose Zoi schemas.
- External/runtime failures should be normalized through `IntentLedger.Error`.
- Runtime persistence is Bedrock-first through `IntentLedger.BedrockStore` and
  `bedrock_job_queue`.

---
> Source: [mikehostetler/intent_ledger](https://github.com/mikehostetler/intent_ledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
