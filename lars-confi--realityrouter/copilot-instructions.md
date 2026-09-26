## realityrouter

> Sticky sessions, MCP/ACP translation, agent fingerprinting


# Multi-agent support

The router has a protocol layer that natively understands the major agentic clients — Zed, Cursor, Continue / VSCodium, OpenClaw, Claude Code, AutoGPT — and adapts routing, state, and tool-calling formats to each.

## Protocol identification

The router automatically identifies which client is calling it. Detection runs in this order:

- **Request body** — an optional `agent_id` field in the JSON payload.
- **Custom header** — `X-Agent-ID`.
- **Standard header** — the `User-Agent` string, which tools like Zed, VS Code, and Cursor send automatically.

If no identification is found, the router uses `default`. You can monitor per-agent success rates in the dashboard.

> [!NOTE]
> Not every client identifies itself. Tools that reach the router through a
> generic SDK — Aider goes via LiteLLM's OpenAI client, for example — send only
> that library's User-Agent, so they appear as `OpenAI/Python …` rather than by
> name. Where a tool lets you set custom headers, sending `X-Agent-ID` gives it a
> stable identity; see [Tool integrations](integrations.md) for the per-tool
> configuration.

> [!INFO]
> **Why this matters.** Per-agent identification feeds into Reality Router™ calibration. The router learns "Zed asks structurally different questions than Cursor, which asks different questions than AutoGPT" and tunes its probabilities accordingly.

## Sticky session state

Multi-turn tool loops are a problem for naive routers: each turn looks like a new query, so consecutive messages could route to different models. The result is "split-brain" behavior — the agent loses context, makes inconsistent decisions, or fails outright.

RealityRouter maintains a sticky-state map: once a conversation begins, it locks that session to a specific model. The lock persists across the entire tool loop.

### How sessions are tracked

- **Explicit session IDs** — clients like OpenClaw and Continue send their own session identifiers, which the router uses directly.
- **Synthesized IDs** — for clients that don't expose session tracking (Zed, Cursor), the router synthesizes a session ID from the User-Agent + conversation fingerprint and maintains state seamlessly.

## Agent-card discovery

For agent-to-agent (A2A) communication, the router exposes an `/.well-known/agent-card.json` endpoint that lets agents like OpenClaw discover the router's capabilities at runtime:

```bash
curl http://localhost:8000/.well-known/agent-card.json
```

```json
{
  "name": "Reality Router",
  "version": "1.0",
  "capabilities": [
    "codebase-edit",
    "filesystem-search",
    "mcp-proxy"
  ],
  "session_tracking": "sticky",
  "protocols": ["openai", "anthropic", "acp"]
}
```

## MCP / ACP translation layer

Different models expose function-calling in different ways: some natively support OpenAI-style `tools` arrays, some use Anthropic-style tool_use blocks, some don't support function-calling at all. The router translates between them automatically.

### Seamless interception

The router intercepts tool calls from clients using the Agent Client Protocol (Zed and similar) and standardizes them into whatever format the target model expects. Your client doesn't need to know which model is on the other end.

### Graceful fallback

If the model the router picks doesn't natively support function-calling, the router **strips the tools from the request** and injects a fallback prompt that describes the available tools in plain text. The model answers; the response flows back through normally. The agent doesn't break.

## Supported clients

These have been tested end-to-end. Anything OpenAI-compatible should work — these are the ones with explicit protocol adapters:

- **Cursor** — OpenAI-compatible base URL, User-Agent identification.
- **Claude Code** — Anthropic-compatible passthrough.
- **Zed** — Agent Client Protocol via the translation layer; synthesized sessions.
- **Continue / VSCodium** — native session tracking.
- **OpenClaw** — agent-card discovery + sticky sessions + tool-use translation.
- **AutoGPT, custom scripts** — any OpenAI-compatible client.

---
> Source: [Lars-confi/RealityRouter](https://github.com/Lars-confi/RealityRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
