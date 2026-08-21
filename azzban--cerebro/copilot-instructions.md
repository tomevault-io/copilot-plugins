## agent-llm

> LLM agent layer patterns, prompt templates, and multi-provider conventions


# Agent / LLM Layer

## Three-Agent Model

| Agent | Role | Port |
|---|---|---|
| **Screener** | Filters market data for tradeable setups | `port.LLM` |
| **Copilot** | Generates trade suggestions with reasoning | `port.LLM` |
| **Reviewer** | Post-trade audit, lesson extraction | `port.LLM` |

Each agent is an independent struct that depends on `port.LLM` — never on a concrete provider.

## Provider Abstraction

All LLM calls go through `port.LLM`. Concrete providers live in `internal/adapter/llm/`:

```go
// internal/port/llm.go
type LLM interface {
    Complete(ctx context.Context, req LLMRequest) (LLMResponse, error)
}
```

To add a new provider (e.g. Gemini Flash), implement `port.LLM` in `internal/adapter/llm/gemini/` — zero changes to agent code.

## Prompt Templates

Prompts live in `internal/agent/prompts/*.tmpl` (Go `text/template`):

```go
// ✅ GOOD — load once at startup, execute per request
var screenerTmpl = template.Must(template.ParseFiles("internal/agent/prompts/screener.tmpl"))
```

- **Never hardcode prompt strings** in Go source files.
- Template data structs are defined in the same `agent` sub-package that uses them.
- System prompts and user prompts are kept in **separate template files**.

## Safety Rules

- Agents are **advisory only** — they produce suggestions, never execute trades directly.
- Every agent output must pass through the **risk gate** before being acted upon.
- Agent calls must respect `ctx` (timeout + cancellation): set a deadline on the context before calling `LLM.Complete`.

```go
ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
defer cancel()
resp, err := a.llm.Complete(ctx, req)
```

## Logging Agent Decisions

All agent decisions (input, provider used, output, latency) must be written to `port.AgentLogStore` for auditability. Never log raw API keys or user secrets.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
