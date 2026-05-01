## anti-api

> Anti-API proxies Antigravity's internal AI models as an Anthropic-compatible API.

# Anti-API Development Guide

## Overview
Anti-API proxies Antigravity's internal AI models as an Anthropic-compatible API.

## Key Files

- `src/main.ts` - CLI entry point
- `src/server.ts` - Hono HTTP server setup
- `src/services/antigravity/chat.ts` - Core chat logic
- `src/proto/encoder.ts` - Protobuf encoding with model selection
- `src/lib/port-finder.ts` - Antigravity port discovery

## Model Selection

Models are specified via `model` parameter in requests. See `MODEL_ENUM` in `encoder.ts` for supported values.

## API Compatibility

Supports `/v1/messages`, `/v1beta/messages`, and `/messages` endpoints for maximum compatibility.

## Running

```bash
bun run src/main.ts start       # Default port 8964
bun run src/main.ts start -v    # Verbose logging
```

---
> Source: [ink1ing/anti-api](https://github.com/ink1ing/anti-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
