## mcp-llm-gateway

> MCP-compatible LLM gateway that proxies completion requests to downstream OpenAI-compatible providers.

# MCP LLM Gateway

MCP-compatible LLM gateway that proxies completion requests to downstream OpenAI-compatible providers.

## When to use this skill

Use this skill when you need to:
- Access multiple LLM providers through one interface
- Proxy LLM requests to different backends
- List available models from remote endpoints

## Tools

- `list_models()` - List available models
- `complete(prompt, model, max_tokens, temperature)` - Send completion request

## Resources

- `models://list` - Available models list
- `config://info` - Gateway configuration

## Configuration

- `DOWNSTREAM_URL` - Base URL for OpenAI-compatible API
- `DEFAULT_MODEL` - Default model name
- `MODEL_LIST_URL` - URL to fetch models list
- `API_KEY` - Optional API key

## Install

```bash
pip install mcp-llm-gateway
```

---
> Source: [daedalus/mcp-llm-gateway](https://github.com/daedalus/mcp-llm-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
