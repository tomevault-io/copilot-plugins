## pulso

> A headless, open-source observability backend: unified logs/metrics/traces access, built-in alerting, and a native Model Context Protocol (MCP) interface, built on Elixir/OTP with a Rust hot path for columnar data.

# Pulso

A headless, open-source observability backend: unified logs/metrics/traces access, built-in alerting, and a native Model Context Protocol (MCP) interface, built on Elixir/OTP with a Rust hot path for columnar data.

Pulso is agent-native from day one. It does not ship a UI. Consumers are humans through their own dashboards and, first-class, AI agents through MCP.

## Read this before any non-trivial design work

**[`docs/architecture.md`](./docs/architecture.md)** — source of truth for Pulso's architecture. Load it before you reason about ingest, storage, coordination, alerting, the Elixir/Rust split, or the MCP boundary. If you're about to add a database, a WAL, a leader-election protocol, or any cluster-visible mutable state, read the "Core bets" and "What Pulso deliberately does not have" sections first.

## Where the project is right now

Early scaffolding. In place:

- Phoenix 1.8 headless app (no HTML, no assets, no Ecto)
- `Pulso.Loki` — read-only Loki HTTP client wrapping `query_range`
- `Pulso.MCP` — JSON-RPC 2.0 dispatcher (`initialize`, `tools/list`, `tools/call`, `ping`)
- `Pulso.MCP.Tools` — tool registry, currently one read-only tool (`query_logs`)
- `PulsoWeb.MCPController` at `POST /mcp` (handles single and batched JSON-RPC)
- `PulsoWeb.OTLPController` at `POST /v1/logs` — OTLP/HTTP JSON logs ingest
- `PulsoWeb.LokiController` at `POST /loki/api/v1/push` — Loki push ingest, JSON and Snappy-compressed protobuf (decoded in Rust by `Pulso.Codec.NIF`)
- `PulsoWeb.CompressedBodyReader` — gzip-aware Plug.Parsers body reader, so JSON receivers accept compressed bodies

Not yet built: alerting, ingestion, Mimir/Tempo clients, storage engine, remediation surface, HITL wiring, distribution (Horde/libcluster/ra).

## Design bet

Grafana's Loki/Mimir/Tempo (and the VictoriaMetrics stack) are already headless, API-only storage — but they're three systems coordinated externally, and existing "AI-ready" MCP servers on top are thin read-only wrappers bolted on after the fact.

Pulso's bet: build the ingest, storage-facade, alerting, and MCP-interface layer as one coherent system, on a runtime (BEAM/OTP) whose concurrency and supervision model fits this problem shape unusually well.

**v1 shape**: MCP-native gateway + shared alerting over proven headless stores (Mimir/Loki/Tempo or VictoriaMetrics underneath). A unified storage engine is a longer-term option once the interface layer proves itself.

## Why Elixir/OTP

- **Per-process resource isolation** — each stream/tenant/query can be its own process with an independent heap and `max_heap_size` cap. Kill a runaway unit of work without taking down the node.
- **Scheduler fairness** — preemptive, reduction-based scheduling; one expensive query cannot easily starve the rest.
- **Backpressure-aware ingestion** — GenStage/Broadway for demand-driven pipelines. Process mailboxes are unbounded by default, so this must be designed for deliberately.
- **Self-healing alerting** — each alert rule as a supervised process; "let it crash and restart" maps directly onto rule-evaluation correctness.
- **Clustering without external coordinators** — distributed Erlang + Horde (CRDT distributed supervisor/registry) for cluster-wide singleton ownership. `libcluster` for discovery.
- **Phoenix PubSub** — distributed pub/sub for cross-signal correlation and alert fan-out.

## Known constraints to design around

- Default distributed Erlang is a full mesh, doesn't scale cleanly past ~100–200 nodes without partitioned topologies.
- Horde's CRDT sync is *eventually consistent* — fine for shard ownership, not strong enough alone for "exactly-once alert firing." Plan to use `ra` (RabbitMQ's Raft, in Erlang) for that specific guarantee.
- Binary sub-references can pin large buffers in memory — `:binary.copy/1` discipline is needed when parsing large payloads and keeping small slices.
- Distributed Erlang's cookie auth is weak by default — cluster must stay inside a VPC, or use TLS distribution.

## MCP read/write boundary (load-bearing rule)

Pulso deliberately splits its MCP surface by risk class:

| Tier | Where it lives | Examples |
|---|---|---|
| Read-only diagnosis | Pulso's MCP server (this repo) | query logs/metrics/traces, correlate signals, fetch alert context |
| Write within Pulso | Pulso's MCP server (this repo) | silence/ack alert, attach annotation |
| Infrastructure remediation | **Separate MCP server, not this repo** | restart service, roll back deploy, scale |

**Do not** add infrastructure remediation tools to `Pulso.MCP.Tools`. Their blast radius must be contained by construction, not by hoping the agent behaves. Anything destructive should be gated through the human-in-the-loop pause that the agent runtime (e.g. Google AX) provides natively.

## Repo layout

```
lib/
  pulso/
    application.ex        # OTP supervision root
    loki.ex               # Read-only Loki HTTP client
    mcp.ex                # JSON-RPC dispatcher (public MCP entry point)
    mcp/
      tools.ex            # Tool registry — read-only tools only
  pulso_web/
    controllers/
      mcp_controller.ex   # POST /mcp — thin JSON-RPC transport
    endpoint.ex
    router.ex
    telemetry.ex
config/                   # Standard Phoenix config; Loki base_url lives here
```

Storage backend URLs are read from `config :pulso, Pulso.Loki, base_url: ...` and analogous config keys for future backends. Never hardcode.

## Conventions

- **HTTP client**: use `Req`. Never `HTTPoison`, `Tesla`, `:httpc`, or `Finch` directly.
- **JSON**: use `Pulso.JSON` (Rust fast path that defers to Elixir's built-in `JSON` for anything it cannot match exactly, so behavior and errors are `JSON`'s), or `JSON` itself where the native code is not loaded yet (`config/runtime.exs`); never `Jason`. Phoenix's `:json_library` is set to `Pulso.JSON` in `config/config.exs`, and a Credo rule (`Credo.Check.Warning.ForbiddenModule`) fails CI on any direct `Jason.*` reference. Jason may still appear as a transitive dep of `phoenix` or a dev dep, but no code in `lib/`, `config/`, or `test/` may call it.
- **Rust fast paths** must be semantically identical to an Elixir reference implementation: when the Rust side cannot guarantee the same result it returns `:fallback` and the Elixir code runs. Tests compare the two on randomized input and assert the Rust path actually answered.
- **New backends** go under `Pulso.<Backend>` (e.g. `Pulso.Mimir`, `Pulso.Tempo`), with the same read-only-first shape as `Pulso.Loki`. Every read function must accept a `:base_url` override in opts.
- **MCP tools** live in `Pulso.MCP.Tools`. Each tool has an `inputSchema`, and its `call/2` clause returns `{:ok, [content_block]}` or `{:error, reason}`. Content blocks follow the MCP shape: `%{"type" => "text", "text" => "..."}`.
- **Alerting** (when added): each rule is its own supervised process, cluster-wide singleton via Horde. Rules that require exactly-once firing route through `ra`.
- **Ingestion** (when added): pull-based via Broadway/GenStage. No unbounded process mailboxes.
- **Naming**: predicate functions end in `?`, not `is_` (see Elixir guidelines below).
- **Rust NIF distribution**: the NIF crates under `native/` (`pulso_object_store`, `pulso_codec`) ship via `rustler_precompiled`. Every `v*` tag triggers `.github/workflows/release.yml`, which builds artifacts for each crate and the target triples in its `lib/pulso/*/nif.ex` module and attaches them to the matching GitHub Release. Downstream consumers install without a Cargo toolchain. Local dev keeps compiling from source (`PULSO_NIF_FORCE_BUILD=true` is the default); unset it to opt into the precompiled path.
- **Memory copies across the NIF boundary**: minimize them. GET streams the S3 body into a Rustler `NewBinary` allocated on the Erlang heap (one copy total, no Rust-side intermediate). PUT does not copy: the Erlang binary is saved into a process-independent `OwnedEnv` and wrapped with `Bytes::from_owner`, so every clone reqwest's retry layer makes shares an owner that keeps the binary alive (a lifetime-extended slice would be unsound for exactly that reason). S3 clients are cached per config so connections are reused. The codec (`pulso_codec`) writes encoder output straight into an Erlang binary and never copies it at the end. The Loki decoder decompresses once into a `NewBinary` and returns every string as a sub-binary of it, and the JSON and segment decoders return strings over 64 bytes as sub-binaries of their input, so retained decoded data pins its source buffer: `:binary.copy/1` anything kept past the request.

## Development workflow

- `mix setup` — fetch deps
- `mix compile --warnings-as-errors` — must be clean before merge
- `mix test` or `mix test test/path/to_test.exs`
- `mix precommit` — runs compile-with-warnings-as-errors, unused-deps check, formatter, tests
- The Elixir/Erlang toolchain is pinned in `mise.toml`; run `mise install` to match.

## Related resources

- Design brief covering Pulso's rationale, alerting model, and agent-integration tiers lives in the project notes (paste from the Claude Desktop session), not in-repo yet.
- Grafana Loki HTTP API — the shape `Pulso.Loki` targets.
- Model Context Protocol spec — `Pulso.MCP` targets the 2025-06-18 protocol version.

---

The sections below are auto-managed framework rules. Preserve the markers.

<!-- usage-rules-start -->

<!-- phoenix:elixir-start -->
## Elixir guidelines

- Elixir lists **do not support index based access via the access syntax**

  **Never do this (invalid)**:

      i = 0
      mylist = ["blue", "green"]
      mylist[i]

  Instead, **always** use `Enum.at`, pattern matching, or `List` for index based list access, ie:

      i = 0
      mylist = ["blue", "green"]
      Enum.at(mylist, i)

- Elixir variables are immutable, but can be rebound, so for block expressions like `if`, `case`, `cond`, etc
  you *must* bind the result of the expression to a variable if you want to use it and you CANNOT rebind the result inside the expression, ie:

      # INVALID: we are rebinding inside the `if` and the result never gets assigned
      if connected?(socket) do
        socket = assign(socket, :val, val)
      end

      # VALID: we rebind the result of the `if` to a new variable
      socket =
        if connected?(socket) do
          assign(socket, :val, val)
        end

- **Never** nest multiple modules in the same file as it can cause cyclic dependencies and compilation errors
- **Never** use map access syntax (`changeset[:field]`) on structs as they do not implement the Access behaviour by default. For regular structs, you **must** access the fields directly, such as `my_struct.field` or use higher level APIs that are available on the struct if they exist, `Ecto.Changeset.get_field/2` for changesets
- Elixir's standard library has everything necessary for date and time manipulation. Familiarize yourself with the common `Time`, `Date`, `DateTime`, and `Calendar` interfaces by accessing their documentation as necessary. **Never** install additional dependencies unless asked or for date/time parsing (which you can use the `date_time_parser` package)
- Don't use `String.to_atom/1` on user input (memory leak risk)
- Predicate function names should not start with `is_` and should end in a question mark. Names like `is_thing` should be reserved for guards
- Elixir's builtin OTP primitives like `DynamicSupervisor` and `Registry`, require names in the child spec, such as `{DynamicSupervisor, name: MyApp.MyDynamicSup}`, then you can use `DynamicSupervisor.start_child(MyApp.MyDynamicSup, child_spec)`
- Use `Task.async_stream(collection, callback, options)` for concurrent enumeration with back-pressure. The majority of times you will want to pass `timeout: :infinity` as option

## Mix guidelines

- Read the docs and options before using tasks (by using `mix help task_name`)
- To debug test failures, run tests in a specific file with `mix test test/my_test.exs` or run all previously failed tests with `mix test --failed`
- `mix deps.clean --all` is **almost never needed**. **Avoid** using it unless you have good reason

## Test guidelines

- **Always use `start_supervised!/1`** to start processes in tests as it guarantees cleanup between tests
- **Avoid** `Process.sleep/1` and `Process.alive?/1` in tests
  - Instead of sleeping to wait for a process to finish, **always** use `Process.monitor/1` and assert on the DOWN message:

      ref = Process.monitor(pid)
      assert_receive {:DOWN, ^ref, :process, ^pid, :normal}

   - Instead of sleeping to synchronize before the next call, **always** use `_ = :sys.get_state/1` to ensure the process has handled prior messages
<!-- phoenix:elixir-end -->

<!-- phoenix:phoenix-start -->
## Phoenix guidelines

- Remember Phoenix router `scope` blocks include an optional alias which is prefixed for all routes within the scope. **Always** be mindful of this when creating routes within a scope to avoid duplicate module prefixes.

- You **never** need to create your own `alias` for route definitions! The `scope` provides the alias, ie:

      scope "/admin", AppWeb.Admin do
        pipe_through :browser

        live "/users", UserLive, :index
      end

  the UserLive route would point to the `AppWeb.Admin.UserLive` module

- `Phoenix.View` no longer is needed or included with Phoenix, don't use it
<!-- phoenix:phoenix-end -->

<!-- usage-rules-end -->

---
> Source: [tuist/pulso](https://github.com/tuist/pulso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
