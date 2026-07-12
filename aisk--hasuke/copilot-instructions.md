## hasuke

> Lightweight Claude agent in idiomatic Haskell.

# hasuke

Lightweight Claude agent in idiomatic Haskell.

## Build & Run

```bash
stack build
ANTHROPIC_API_KEY=... stack run
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | one of two | API key auth |
| `ANTHROPIC_AUTH_TOKEN` | one of two | Bearer token auth |
| `ANTHROPIC_BASE_URL` | no | Override base URL (default: `https://api.anthropic.com`) |

## Project Structure

```
app/Main.hs       -- Entry point: load env → chatLoop
src/Types.hs      -- Core types: Role, Message, ToolCall, LLMResponse, Credentials, Session
src/Tool.hs       -- ToolDef, ToolRegistry, built-in tools (read_file, write_file, exec)
src/Provider.hs   -- LLMProvider typeclass + AnthropicProvider
src/Agent.hs      -- Agent record + runTurn loop
```

## Key Design Notes

- `Message.content :: Value` — supports both plain `String "text"` and structured content blocks (required for Anthropic tool_use/tool_result)
- HTTP requests use `send` (not `post`) to attach custom headers (`x-api-key`, `anthropic-version`)
- Tool schemas use `"input_schema"` key (Anthropic format, not `"parameters"`)
- `DuplicateRecordFields` + `OverloadedRecordDot` throughout — always use dot notation (`x.field`)

## GHC Extensions Used

`DuplicateRecordFields`, `OverloadedRecordDot`, `OverloadedStrings`

---
> Source: [aisk/hasuke](https://github.com/aisk/hasuke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
