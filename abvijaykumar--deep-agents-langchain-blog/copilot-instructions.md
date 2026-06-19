## deep-agents-langchain-blog

> This project demonstrates the "Deep Agent" architecture using LangChain and the Agent Skills specification.

# Project Context: Deep Agents Demo

## Core Mission
This project demonstrates the "Deep Agent" architecture using LangChain and the Agent Skills specification.

## Conventions
- Use functional programming patterns for tools.
- Skills should be stored as directories under `skills/`.
- Every skill must have a `SKILL.md` file.
- Subagents should be used for complex, multi-step tasks to isolate context.
- All intermediate work should be saved to the `./workspace` directory.

## Known Entities
- **Main Agent**: The orchestrator who plans and delegates.
- **Researcher**: Specialized in web search and data aggregation.
- **Writer**: Specialized in formatting and synthesizing information into reports.

---
> Source: [abvijaykumar/deep-agents-langchain-blog](https://github.com/abvijaykumar/deep-agents-langchain-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
