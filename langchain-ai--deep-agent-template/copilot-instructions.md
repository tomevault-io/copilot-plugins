## deep-agent-template

> All agents are defined in `src/deep_agent/graph.py`.

# Agents

All agents are defined in `src/deep_agent/graph.py`.

## Conventions

- Prefer async-native code wherever possible for performance. Tools, tests, and any new I/O should use async.
- New tools should be low-dependency and safe to run on a remote server.
- This deploys in a web server. Avoid calls to actual file system.

---
> Source: [langchain-ai/deep-agent-template](https://github.com/langchain-ai/deep-agent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
