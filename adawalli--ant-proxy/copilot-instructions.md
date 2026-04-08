## ant-proxy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

Claude Gateway is an Anthropic-to-OpenAI API proxy. It accepts requests in Anthropic `/v1/messages` format, translates them to OpenAI `/v1/chat/completions` format, forwards them to a configurable backend (OpenRouter or Zen), and translates responses back. Supports both streaming and non-streaming requests.

## Commands

```bash
bun install          # Install dependencies
bun run dev          # Start with --watch (auto-reload)
bun run start        # Start without watch
bun test             # Run all tests
bun test tests/anthropic.test.ts  # Run a single test file
```

Use with Claude Code: `ANTHROPIC_BASE_URL=http://localhost:8082 claude --model qwen3-coder`

## Architecture

All source code is in `src/`, tests in `tests/`.

- `src/index.ts` - Entry point. Bun.serve() HTTP server on PORT (default 8082). Handles `/v1/messages` (POST) and `/v1/models` (GET). Contains streaming and non-streaming request handling, cost estimation, and tracing span management.
- `src/lib/anthropic.ts` - Type definitions for both Anthropic and OpenAI request/response formats. `convertToOpenAI()` and `convertFromOpenAI()` handle the bidirectional translation, including tool_calls mapping.
- `src/lib/streaming.ts` - `convertToAnthropicStream()` converts OpenAI SSE stream chunks to Anthropic stream event format. `createSSEEvent()` serializes events to SSE wire format.
- `src/lib/router.ts` - `getRouterConfig()` reads env vars to determine which backend to use. Returns base URL and API key for either OpenRouter or Zen.
- `src/lib/phoenix.ts` - Optional OpenTelemetry tracing via Phoenix/Arize. NoOpSpan pattern when tracing is disabled (no PHOENIX_ENDPOINT set).

## Key Design Decisions

- Bun runtime, not Node. Use `bun` for everything (see project CLAUDE.md conventions).
- No frameworks - raw `Bun.serve()` with manual routing.
- Tracing is opt-in: the NoOpSpan class avoids conditionals throughout the codebase when Phoenix isn't configured.
- Env vars are loaded automatically by Bun (no dotenv).

## Environment Variables

Copy `.env.example` to `.env`. Required: either `OPENROUTER_API_KEY` or `ZEN_API_KEY`.

| Variable             | Description                                      | Default          |
| -------------------- | ------------------------------------------------ | ---------------- |
| `PORT`               | Server port                                      | `8082`           |
| `ROUTER_PROVIDER`    | Backend: `openrouter` or `zen`                   | `openrouter`     |
| `OPENROUTER_API_KEY` | Your OpenRouter API key                          | -                |
| `ZEN_API_KEY`        | Your Zen API key                                 | -                |
| `ANTHROPIC_API_KEY`  | Direct Anthropic passthrough (for Claude models) | -                |
| `PHOENIX_ENDPOINT`   | Phoenix OTLP endpoint (tracing)                  | -                |
| `PHOENIX_PROJECT`    | Phoenix project name                             | `claude-gateway` |
| `REQUEST_TIMEOUT_MS` | Non-streaming request timeout                    | `120000`         |
| `STREAM_TIMEOUT_MS`  | Streaming request timeout                        | `300000`         |
| `DRAIN_TIMEOUT_MS`   | Graceful shutdown drain timeout                  | `30000`          |
| `LOG_LEVEL`          | Logging level                                    | `info`           |

## Health Endpoints

- `/health` - Liveness probe (always 200 if running)
- `/ready` - Readiness probe (checks API key + upstream connectivity)
- `/healthz` - Detailed status (uptime, memory, request counts, upstream info)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adawalli)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adawalli)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
