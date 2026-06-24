## trinity-coordinator

> - `lib/` contains Trinity coordinator runtime modules and Mix tasks.

# Repository Guidelines

## Project Structure
- `lib/` contains Trinity coordinator runtime modules and Mix tasks.
- `test/` contains ExUnit coverage; process-env mutation belongs only in tests.
- `examples/`, `guides/`, `docs/`, and `README.md` must stay aligned with task options and dependency-source behavior.
- `docs/priv/` is local/private/generated material and is not packaged.

## Dependency Sources
- Dependency source selection is handled by `build_support/dependency_sources.exs` and `build_support/dependency_sources.config.exs`.
- Local dependency overrides use `.dependency_sources.local.exs`.
- Dependency source selection must not use environment variables.
- Keep internal dependency declarations in the manifest instead of adding one-off path/git resolver logic to `mix.exs`.

## Runtime Env
- Runtime application code under `lib/**` must not call direct OS env APIs such as `System.get_env`, `System.fetch_env`, `System.put_env`, or `System.delete_env`.
- Runtime/deployment env reads belong in `config/runtime.exs` or a `Config.Provider`.
- Mix tasks must accept explicit flags for provider gates and credentials instead of reading process env directly.
- Library APIs receive explicit options, config structs, application config materialized by the top-level app, or caller-supplied env maps.
- Tests may manipulate env only for config-boundary, live-gate, or compatibility checks.

## Gates
- Run `mix format`.
- Run `mix compile --warnings-as-errors`.
- Run `mix test`.
- Run `mix credo --strict`.
- Run `mix dialyzer`.
- Run `mix docs --warnings-as-errors`.
- Run `mix hex.build --unpack` when package files change, then remove the unpacked directory.

---
> Source: [nshkrdotcom/trinity_coordinator](https://github.com/nshkrdotcom/trinity_coordinator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
