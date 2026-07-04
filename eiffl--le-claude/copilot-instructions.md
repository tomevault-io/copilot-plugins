## le-claude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

le-claude is a local HTTP proxy that translates between Anthropic's Messages API format (used by Claude Code) and OpenAI-compatible Chat Completions format (used by France's Albert API). It has **zero npm dependencies** — only Node.js builtins (>=22).

## Commands

- **Run tests:** `npm test`
- **Run a single test file:** `node --test test/test-translate.js`
- **Run a single named test:** `node --test --test-name-pattern="pattern" test/test-translate.js`
- **Run locally:** `node bin/le-claude` (requires prior `--setup` for API key config)

## Architecture

The proxy intercepts Claude Code's Anthropic API calls and translates them to OpenAI format for Albert:

```
Claude Code → POST /v1/messages → proxy (127.0.0.1:random) → POST /v1/chat/completions → Albert API
```

**Key modules in `src/`:**

- **cli.js** — Entry point. Parses args (including `--model` flag), loads config, starts proxy, spawns Claude Code as child process with `ANTHROPIC_BASE_URL` pointing to the local proxy, handles shutdown signals.
- **config.js** — Reads/writes `~/.config/le-claude/config.json` (API key, model, base URL). Interactive setup flow fetches available models from Albert.
- **proxy.js** — HTTP server on localhost. Handles `/v1/messages` (main proxy) and `/v1/messages/count_tokens` (stub). Supports both streaming and non-streaming responses.
- **translate.js** — Pure functions for format conversion. Request: Anthropic content blocks → OpenAI messages, tool definitions, tool_choice. Response: OpenAI completion → Anthropic message format. Strips thinking blocks, filters BatchTool, removes `format: "uri"` from schemas.
- **stream.js** — `StreamTranslator` class: stateful translator that processes OpenAI SSE chunks incrementally and emits Anthropic SSE events. Tracks content block indices and accumulates tool call arguments across chunks.
- **server-tools.js** — Handles Anthropic server-side tools (web_search, web_fetch, code_execution). Detects them in requests, converts to function tools for the backend, executes search/fetch locally (Marginalia by default, Brave Search if configured), and runs a multi-turn loop to feed results back to the model.

## Key Design Decisions

- Proxy binds to `127.0.0.1` only (never network-exposed) on a random port (port 0).
- `translate.js` is side-effect free — all translation logic is pure and directly unit-testable.
- `StreamTranslator` is stateful by necessity — it maintains block index mappings and accumulated tool arguments across SSE chunks.
- API keys stored with `0o600` permissions. Debug logging truncates message content and omits keys.
- Token counting endpoint returns an estimate (text length / 4) since Albert doesn't expose token counting.
- Server-side tools (web_search, web_fetch) are intercepted by the proxy, executed locally, and results are fed back to the model in a multi-turn loop. code_execution is stripped (Claude Code's Bash tool covers this). Web search defaults to Marginalia (Swedish, EU-funded, zero config), with optional Brave Search API for better results.

## Testing

Tests use Node.js built-in `node:test` module and `node:assert`. Three test files:

- **test-translate.js** — Unit tests for pure translation functions
- **test-stream.js** — StreamTranslator chunk processing
- **test-server-tools.js** — Server tool detection, partitioning, function conversion, block injection, and history translation
- **test-integration.js** — Spins up the proxy against a mock OpenAI backend, tests full request/response cycle including streaming, error handling, server tool handling, and edge cases

---
> Source: [EiffL/le-claude](https://github.com/EiffL/le-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
