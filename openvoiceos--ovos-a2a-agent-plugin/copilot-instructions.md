## ovos-a2a-agent-plugin

> Delegates OVOS persona conversations to any external agent that speaks the

# ovos-a2a-solver-plugin — Agent Skill Card

## What this plugin does

Delegates OVOS persona conversations to any external agent that speaks the
[Agent2Agent (A2A)](https://google.github.io/A2A/) protocol.

A2A agents expose a discovery card at `/.well-known/agent.json` and accept
tasks as JSON-RPC 2.0 requests. This plugin bridges the gap between the OVOS
persona pipeline and any A2A-compliant agent — local or remote.

## Accessibility angle

Voice-only and blind users interact with OVOS personas that may not have
built-in knowledge of every domain. By pointing a persona at a specialist
A2A agent (e.g. a medical Q&A agent, a transit planner, a recipe assistant),
the persona gains that domain without any code changes. The user speaks
naturally; the system routes silently.

## Quick config (persona YAML)

```yaml
name: my-specialist-persona
engine: ovos-a2a-solver
engine_config:
  agent_url: "https://my-a2a-agent.example.com"
  auth_header: "Bearer <token>"   # optional
  timeout: 60
  streaming: false
```

## Key methods

| Method | What it does |
|--------|-------------|
| `A2AClient.fetch_agent_card()` | Fetch and parse `/.well-known/agent.json` |
| `A2AClient.send_task(text, session_id, history)` | Blocking task submission |
| `A2AClient.stream_task(text, session_id, history)` | SSE streaming task submission |
| `A2AChatEngine.continue_chat(messages, session_id)` | OPM ChatEngine entry point |
| `A2AChatEngine.stream_tokens(messages, session_id)` | Streaming OPM entry point |

## How to speak results

The plugin returns the agent's plain-text response directly. No special
formatting is applied — the OVOS TTS pipeline speaks the text as-is.

---
> Source: [OpenVoiceOS/ovos-a2a-agent-plugin](https://github.com/OpenVoiceOS/ovos-a2a-agent-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
