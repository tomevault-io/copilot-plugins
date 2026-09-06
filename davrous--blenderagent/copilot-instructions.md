## blenderagent

> This project was built with the microsoft-foundry skill. Before working on or answering questions about Foundry agents, read the microsoft-foundry skill first.

# Repository guidance

## Microsoft Foundry

This project was built with the microsoft-foundry skill. Before working on or answering questions about Foundry agents, read the microsoft-foundry skill first.

The container composes Responses, Activity, and `invocations_ws` protocols on one Agent Server host. Preserve cooperative host inheritance and register voice through the invocations SDK's `ws_handler`.

Typed and voice turns share one Foundry Responses conversation in hosted mode. Keep `agent_session_id` for sandbox affinity and `conversation` for transcript continuity; they are not interchangeable.

## Focused checks

```bash
python devTools/test_voice_pipeline.py
python devTools/test_activity_history.py
cd webchat && npm run build
node ../devTools/test_voice_relay.mjs
```

---
> Source: [davrous/blenderagent](https://github.com/davrous/blenderagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
