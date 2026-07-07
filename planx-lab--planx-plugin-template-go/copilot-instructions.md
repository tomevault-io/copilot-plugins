## planx-plugin-template-go

> 1. [planx-architecture.md](../planx-architecture.md)

# AI RULES — PLANX GO CONNECTOR (v4)

## Authority Documents

1. [planx-architecture.md](../planx-architecture.md)
2. [planx-ai-guardrails.md](../planx-ai-guardrails.md)
3. [AI_CONTRACT.md](../AI_CONTRACT.md)
4. [planx-sdk-go/AI.md](../planx-sdk-go/AI.md) — SDK semantics

---

## SCOPE
This repository implements Connector business logic. A Connector is ONE plugin
binary that MAY expose multiple Components (Source, Processor, Sink) — ADR-009.

---

## CONNECTOR HARD RULES

AI MUST NOT:
- Implement gRPC servers
- Manage sessions or flow control
- Start goroutines for concurrency
- Import planx-engine
- Import planx-proto directly
- Read from STDIN or write to STDOUT (except logging)

AI MUST:
- Place each component kind under its own directory:
  - `internal/source/` — implements `sdk.SourceSPI`
  - `internal/processor/` — implements `sdk.ProcessorSPI`
  - `internal/sink/` — implements `sdk.SinkSPI`
- Have exactly ONE binary: `cmd/plugin/main.go` calls `sdk.Serve(sdk.Plugin{...})`
  declaring one or more `sdk.ComponentSpec`s (multi-component, ADR-009).
- Implement SPI interfaces from `planx-sdk-go/sdk` (NOT a `spi` package).
- Keep logic synchronous and deterministic.
- Treat Batch as opaque bytes.
- NOT ship a `manifest.yaml` — the binary is self-describing via `Discover`
  (ADR-008).

If a requirement seems to need runtime logic:
STOP. That belongs to SDK.

---
> Source: [planx-lab/planx-plugin-template-go](https://github.com/planx-lab/planx-plugin-template-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
