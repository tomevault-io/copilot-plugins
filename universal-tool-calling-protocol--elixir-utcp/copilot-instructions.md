## elixir-utcp

> - **Full verification (CI)**: `mix verify` — runs compile, format check, credo, dialyzer, tests with coverage, docs warnings

# AGENTS.md

## Commands

- **Full verification (CI)**: `mix verify` — runs compile, format check, credo, dialyzer, tests with coverage, docs warnings
- **Run unit tests** (default, excludes integration): `mix test`
- **Run all tests including integration**: `mix test --include integration`
- **Run integration tests only**: `mix test --only integration`
- **Run a single test file**: `mix test path/to/test.exs`
- **Lint**: `mix lint` — format check + credo + dialyzer
- **Lint fix**: `mix lint.fix` — format + credo
- **Pre-commit check**: `mix precommit` — compile w/ warnings-as-errors, check unused deps, format
- **Format**: `mix format` (uses Quokka plugin via `.formatter.exs`)
- **Type check**: `mix dialyzer`
- **Security scan**: `mix sobelow` (config in `.sobelow-conf`)
- **Generate docs**: `mix docs`

## Dialyzer

- PLT stored at `priv/plts/dialyzer.plt` (gitignored, cached in CI).
- First run builds PLT from scratch; subsequent runs are incremental.
- Flags: `:error_handling`, `:underspecs`.

## Testing Gotchas

- Integration tests are **excluded by default** (`ExUnit.start(exclude: [:integration])` in `test/test_helper.exs`). External services required.
- Some tests use `@tag :skip` intentionally (WebRTC, TCP/UDP mock, monitoring/performance).
- Mox mocks are defined in `test/test_helper.exs` and `test/support/mocks.ex`.
- Each transport has multiple test variants: `*_mox_test.exs`, `*_mox_simple_test.exs`, `*_test.exs` (integration), `*_unit_test.exs`.

## Architecture

- **Entry point**: `ExUtcp.Client` (GenServer, `lib/ex_utcp/client.ex`)
- **Config**: `ExUtcp.Config` — no `config/` directory or mix config; all runtime via `Config.new/1`
- **8 transports**: HTTP, CLI, WebSocket, gRPC, GraphQL, MCP, TCP/UDP, WebRTC — each in `lib/ex_utcp/transports/<name>.ex` with subdirectory for supporting modules
- **Transport behaviour**: `lib/ex_utcp/transports/behaviour.ex`
- **gRPC protobuf**: Definitions in `proto/utcp.proto`, generated Elixir modules in `lib/ex_utcp/grpcpb/`
- **Core modules**: Client, Config, Providers, Tools, Repository, Search, Monitoring, OpenApiConverter

## Style Conventions

- Formatter uses **Quokka** plugin (not default Elixir formatter alone).
- Credo runs in strict mode with cyclomatic complexity max 15 and max nesting 4.
- Max line length: 120 (low priority warning).

## Security

- Sobelow suppressions in `.sobelow-conf` are known and mitigated:
  - Directory traversal in `client.ex:252` and `openapi_converter.ex:106` — path validation in place
  - Command injection in `transports/cli.ex:96,132` — command path validation in place
  - `String.to_atom` DOS in `websocket.ex:274` and `websocket/testable.ex:317` — uses `String.to_existing_atom`

## License

- `mix.exs` declares **MPL-2.0**; README says MIT. Verify before publishing.

---
> Source: [universal-tool-calling-protocol/elixir-utcp](https://github.com/universal-tool-calling-protocol/elixir-utcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
