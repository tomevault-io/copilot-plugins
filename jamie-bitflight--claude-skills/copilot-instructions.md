## json-no-pretty-print

> Do not pretty-print JSON by default for machine or LLM consumption


# JSON output (no pretty-print by default)

When emitting JSON for **scripts, tools, MCP, or any path where output is consumed by programs or LLMs** (not a human reading a terminal for debugging):

- Use **`JSON.stringify(value)`** only — do **not** use `JSON.stringify(value, null, 2)` or other spacing/indent by default.
- Extra newlines and indentation are **token cost** for models and add no parsing benefit.

---
> Source: [Jamie-BitFlight/claude_skills](https://github.com/Jamie-BitFlight/claude_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
