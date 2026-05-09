## core-agent

> Build the main LangChain agent orchestrating all tools and workflows:

# 🧠 Task: Core LangChain Agent for myquery

## 🎯 Goal
Build the main LangChain agent orchestrating all tools and workflows:
- Connect DB
- Get schema
- Generate query
- Execute query
- Visualize
- Analyze data
- Handle multi-database sessions

## ⚙️ Requirements
- Create `core/agent.py`.
- Use LangChain’s `initialize_agent` with `Tool` objects.
- Register all tools dynamically.
- Maintain conversational memory (schema + recent queries).
- Handle debug and safe execution modes.

## 🧠 Steps
1. Define `load_tools()` function to load from `/tools`.
2. Define `create_agent()` to initialize LangChain agent.
3. Implement `run_agent(prompt)` entry function.
4. Integrate MCP context (port 7766).
5. Handle logging and output formatting centrally.

## 🧩 Cursor Hints
- Use `@cursor: define-agent myquery_agent`
- Add structured logging (schema, query, execution time).
- Ensure agent is stateless between sessions unless memory is active.

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
