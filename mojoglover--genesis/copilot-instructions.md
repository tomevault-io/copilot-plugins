## genesis

> **Agents are fully self-contained.** No agent links to outside entities for internal processes.

# GENESIS — Claude Rules

## Agent Architecture: Self-Contained Modules

**Agents are fully self-contained.** No agent links to outside entities for internal processes.

- All internal logic (memory, planning, execution, RAG, tools) runs locally within the agent
- Agents do NOT make direct external API calls from internal modules
- **Any connection to the outside world goes through `plugops_bridge` (stdio)**
- plugops_bridge is the single defined IO channel for external communication
- If an agent needs external data, it requests it through plugops_bridge — not directly

This applies to every agent forged from BlackZero. Enforce it in config.yaml and during forge.

## No Silent Placeholders

**Every placeholder file must make its incompleteness loud.**

- Python stubs must raise `NotImplementedError` with a descriptive message
- Never leave a function body as `pass` or `return None` silently
- Never create an empty file and commit it as "scaffolding"

Examples:
```python
# GOOD
def process(self, task):
    raise NotImplementedError("process_architect_0: process() not implemented — wire workflow engine here")

# BAD
def process(self, task):
    pass
```

This applies to all agents, modules, tools, and subsystems forged from BlackZero.

## Branch Policy

We use `main`. Always.

---
> Source: [MojoGlover/Genesis](https://github.com/MojoGlover/Genesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
