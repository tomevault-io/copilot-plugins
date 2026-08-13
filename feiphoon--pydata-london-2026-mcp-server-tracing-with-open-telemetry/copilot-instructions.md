## pydata-london-2026-mcp-server-tracing-with-open-telemetry

> ACCEPT_EULA=true docker compose up --build

# agents.md — Project TANUKI: environment setup guide

## Bring the full stack up

```bash
ACCEPT_EULA=true docker compose up --build
```

This starts all services defined in `docker-compose.yml` and `docker-compose.override.yml`. The first run takes a few minutes while images are pulled and the two local images (`nook-mcp`, `nook-seeder`) are built.

## Services

| Service         | URL / address                   | Credentials       | Purpose                                         |
| --------------- | ------------------------------- | ----------------- | ----------------------------------------------- |
| Lenses HQ       | http://localhost:9991           | `admin` / `admin` | Control plane, OAuth issuer (used in section 7) |
| nook-mcp        | http://localhost:8000/mcp       | —                 | Workshop MCP server                             |
| Jaeger UI       | http://localhost:16686          | —                 | Trace visualisation                             |
| OTel Collector  | OTLP gRPC `:4317`, HTTP `:4318` | —                 | Receives spans from `nook-mcp`                  |
| Kafka (host)    | `localhost:9092`                | —                 | Broker, accessible from the host                |
| Schema Registry | http://localhost:8081           | —                 | Available; not used           |

## Kafka topics

The `nook-seeder` container creates and continuously populates these topics:

| Topic                 | Partitions | Rate               | Notes                                                      |
| --------------------- | ---------- | ------------------ | ---------------------------------------------------------- |
| `home-upgrade-quotes` | 1          | write-only via MCP | Created by seeder; records written by `publish_event` tool |
| `abd-balance`         | 1          | 1 msg/s            | Player balance updates, with occasional suspicious deposits; feeds ABD multiplier |
| `catch-log`           | 1          | 1 msg / 5 min      | Fish/bug catches; feeds catch-value multiplier             |
| `island-visitors`     | 1          | 0.1 msg/s          | Flick/CJ triggers 1.5× catch sell-price multiplier         |

## Smoke tests

### 1. Topics are being populated

```bash
docker compose exec demo-kafka kafka-topics \
  --bootstrap-server localhost:9092 --list
```

Expected topics: `abd-balance`, `catch-log`, `home-upgrade-quotes`, `island-visitors`.

Sample records from `abd-balance`:

```bash
docker compose exec demo-kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic abd-balance --max-messages 3 --from-beginning
```

### 2. nook-mcp is responding

Run `smoke_test.py` from the repo root (requires Python 3.12+ with `uv`):

```bash
uv run --with fastmcp==3.2.3 python smoke_test.py
```

Expected output lists the active tools, the active topics, and a Bells quote.

### 3. Traces are flowing into Jaeger

Open <http://localhost:16686>, select `nook-mcp` from the **Service** dropdown, optionally filter by operation `tools/call quote_upgrade`, and click **Find Traces**.

A single `quote_upgrade` call produces a multi-span waterfall:

The six sub-tool spans fan out concurrently via `asyncio.gather`, so
their order shifts between runs; each Kafka `receive` sits under the
sub-tool that triggered it:

| Layer       | Span name                                  | Source                           |
| ----------- | ------------------------------------------ | -------------------------------- |
| MCP         | `tools/call quote_upgrade`                 | FastMCP auto-instrumentation     |
| Application | `tool quote_upgrade`                       | `nook_mcp.tools`                 |
| Application | `tool get_todays_catch`                    | `nook_mcp.tools`                 |
| Messaging   | `catch-log receive`                        | `nook_mcp.kafka_client`          |
| Application | `tool get_plea_fee_multiplier`             | `nook_mcp.tools`                 |
| Application | `tool get_abd_balance`                     | `nook_mcp.tools`                 |
| Messaging   | `abd-balance receive`                      | `nook_mcp.kafka_client`          |
| Application | `tool apply_catch_value_multiplier`        | `nook_mcp.tools`                 |
| Messaging   | `island-visitors receive`                  | `nook_mcp.kafka_client`          |
| Application | `tool get_luck_multiplier`                 | `nook_mcp.tools`                 |
| Application | `tool apply_wealth_potential_multiplier`   | `nook_mcp.tools`                 |
| GenAI       | `chat qwen3.5:0.8b`                        | `nook_mcp.tools._llm_reasoning` / `._simulate_llm_…` |

## Tear down

```bash
docker compose down        # stop; keep volumes
docker compose down -v     # stop and wipe all data
```

## LLM and Chat Interface

Once the Docker stack is up, the LLM and chat interface needs to be set up and connected to the Nook MCP server. You can use frontier lab tools such as Anthropic's Claude Code or OpenAI's Codex. For the local option (Ollama and OpenCode), follow the instructions in [SETUP_LLM_AND_CHAT.md](./SETUP_LLM_AND_CHAT.md)

---
> Source: [feiphoon/pydata-london-2026-mcp-server-tracing-with-open-telemetry](https://github.com/feiphoon/pydata-london-2026-mcp-server-tracing-with-open-telemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
