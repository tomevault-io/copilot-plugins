## skaden

> - Preserve the product boundary: Skaden is a standalone conversational terminal agent whose quantum authority remains deterministic. Other coding agents are design references, not runtime hosts or integration surfaces.

# Skaden contributor instructions

- Preserve the product boundary: Skaden is a standalone conversational terminal agent whose quantum authority remains deterministic. Other coding agents are design references, not runtime hosts or integration surfaces.
- The model may explain, plan, and select safe tools. It may not bypass typed verification, policy, digests, approval, or credential boundaries.
- Model engines are replaceable components inside Skaden. OpenAI, Anthropic, Ollama, and private compatible endpoints must share the same deterministic tool and safety boundary.
- Skaden is the MCP client/host; do not add an outward MCP server for other coding agents.
  Treat imported MCP servers and tools as untrusted, and never let them mint approval or become a
  provider-submission boundary.
- Simulator behavior is the default. Hardware remains disabled until the broker threat model and fault-injection suite exist.
- All persisted records use strict Pydantic schemas and content digests. New pass/fail decisions require deterministic evidence.
- Run `pytest`, `ruff check .`, and `python -m compileall -q src` before handing off code changes.

---
> Source: [SkadenAI/skaden](https://github.com/SkadenAI/skaden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
