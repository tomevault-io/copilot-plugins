## observability

> Observability and logging


# Observability Rules

All errors must contain:
- timestamp
- correlationId
- route
- executionTime
- stacktrace
- request payload

Use:
- structured logs
- OpenTelemetry
- Serilog

All critical operations:
- must be traceable
- must generate logs
- must support debugging

Scheduling logs must contain:
- scheduled arena time
- actual execution time
- timezone
- retry attempts
- skipped executions
- duplicate prevention decisions

Never:
- swallow exceptions
- generate generic logs
- log without context

---
> Source: [davidzaque-leal/royal-arena](https://github.com/davidzaque-leal/royal-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
