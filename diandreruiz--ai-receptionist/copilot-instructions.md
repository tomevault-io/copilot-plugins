## ai-receptionist

> TDD, pytest, and what agent tests must cover — when editing src/agent or tests


# Testing agent behavior (required)

Behavior = code. **Do not ship** new or materially changed agent behavior **without tests**, unless the user explicitly opts out (warn about production regressions).

- **Run:** `uv run pytest` — patterns in **`tests/`**; APIs in [testing docs](https://docs.livekit.io/agents/start/testing/) (confirm with MCP / `lk docs`).
- **TDD:** For **instructions, tool descriptions, tasks, handoffs, workflows** — write tests for intended behavior **first**, then implement until green. Same for **new tools**.

## Layout

```
src/agent.py
tests/test_agent.py   # or more modules under tests/
```

## Cover at least

- Basic conversation / greeting where relevant.  
- **Tools:** right tool and arguments.  
- **Response quality** for representative inputs (where assertable).  
- **Errors / edge cases:** garbage input, silence, failures.  
- **Workflows:** handoffs/tasks when expected; interruptions if the framework allows asserting them.

Find current APIs with e.g. `lk docs search "livekit agents testing"` or MCP.

## Why tests matter

Prompts, tool descriptions, and model updates change behavior silently. Manual “sounds fine” is weak for voice.

## If the user refuses tests

Proceed only if explicit. Recommend tests before production; you may say: *Built without tests as requested — strongly recommend tests before production; voice agents are hard to verify manually.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DiandreRuiz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DiandreRuiz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
