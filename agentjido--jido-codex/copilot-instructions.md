## jido-codex

> `jido_codex` is a harness-first adapter wrapping `codex_sdk` to implement `Jido.Harness.Adapter`.

# AGENTS.md — Jido.Codex

## Overview

`jido_codex` is a harness-first adapter wrapping `codex_sdk` to implement `Jido.Harness.Adapter`.

## Key Files

- `lib/jido_codex.ex` — public API (`run/2`, `run_request/2`, `cancel/1`)
- `lib/jido_codex/adapter.ex` — `Jido.Harness.Adapter` implementation
- `lib/jido_codex/mapper.ex` — Codex SDK event mapping to `Jido.Harness.Event`
- `lib/jido_codex/options.ex` — metadata/runtime normalization and precedence handling
- `lib/jido_codex/compatibility.ex` — exec/app-server compatibility checks

## Commands

- `mix test` — Run tests
- `mix quality` — Full quality check (warnings-as-errors, format, credo, dialyzer, doctor)
- `mix codex.install` — CLI discoverability check
- `mix codex.compat` — transport compatibility diagnostics
- `mix codex.smoke` — minimal runtime smoke execution

## Conventions

- Follow standard Elixir conventions
- Keep adapter scope harness-first (no broad management wrappers)
- Preserve module namespace as `Jido.Codex.*`
- Keep adapter/runtime code under `lib/jido_codex/`

---
> Source: [agentjido/jido_codex](https://github.com/agentjido/jido_codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
