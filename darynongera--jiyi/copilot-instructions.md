## jiyi

> Agent-focused guidance for working on Jiyi.

# AGENTS.md

Agent-focused guidance for working on Jiyi.

## Project overview

Jiyi is an Elixir service that provides durable, vector-backed memory for autonomous agents. It stores episodic events, semantic facts, and per-session working memory, and exposes HTTP and MCP interfaces.

## Setup commands

- Install deps: `mix deps.get`
- Compile: `mix compile`
- Create database: `mix ecto.create`
- Run migrations: `mix ecto.migrate`
- Start service: `mix run --no-halt`

## Test commands

- Full suite: `mix test`
- Non-DB smoke tests: `mix test --no-start test/jiyi_test.exs test/jiyi/api/router_test.exs test/jiyi/retrieval_test.exs`
- Format check: `mix format --check-formatted`

## Code style

- Follow the surrounding code style; no trailing whitespace.
- Do not add comments unless explicitly requested.
- Prefer pattern matching and `with` blocks over nested `case` expressions.
- Use `utc_datetime_usec` for all persisted timestamps.
- Keep GenServer call handlers synchronous; spawn work via `Task.Supervisor`.

## Architecture conventions

- `lib/jiyi/application.ex` owns the top-level supervision tree. Do not change the shape without checking the implementation spec.
- `Jiyi.Repo` uses a custom `Jiyi.PostgrexTypes` module for `pgvector` support.
- Memory stores (`EpisodicStore`, `SemanticStore`, `Quarantine`) are GenServers fronting Ecto.
- `Jiyi.Retrieval` is a plain module; it must tolerate missing or crashed store processes.
- `Jiyi.API.Router` and `Jiyi.API.MCPServer` must accept identical payloads.

## Testing instructions

- Add or update ExUnit tests for any logic change.
- DB-backed tests `use Jiyi.DataCase` and rely on `Ecto.Adapters.SQL.Sandbox`.
- Run `mix test` before finishing; fix failures.
- If the test database is unavailable, at least run the non-DB smoke tests.

## Security considerations

- Never log or commit `JIYI_API_TOKEN`.
- Treat `external_untrusted` provenance as potentially hostile: it goes to `Quarantine`, not the main tables.
- `Jiyi.Anomaly.Watcher` scans for prompt-injection-like phrasing; update its pattern list deliberately.
- `Jiyi.API.Router` requires a bearer token on all HTTP routes except `/mcp`, which is forwarded to the MCP server. MCP tool calls authenticate per-call via `session_token` inside the tool arguments (`Jiyi.Auth.authenticate_mcp/2`); `initialize` / `tools/list` are therefore reachable without a bearer token and this is accepted.

---
> Source: [DarynOngera/jiyi](https://github.com/DarynOngera/jiyi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
