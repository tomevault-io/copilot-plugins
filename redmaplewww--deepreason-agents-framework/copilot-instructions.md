## deepreason-agents-framework

> This project uses OpenCode as the agent foundation. Do not continue the old local Python agent framework as the primary runtime unless the user explicitly asks for legacy migration.

# OpenCode Agent Stack

This project uses OpenCode as the agent foundation. Do not continue the old local Python agent framework as the primary runtime unless the user explicitly asks for legacy migration.

## Architecture Direction

- Prefer OpenCode native capabilities before writing framework code: agents, subagents, permissions, commands, skills, MCP, sessions, export/import, `run`, `serve`, and snapshots.
- Treat Codex CLI, Claude Code/OpenClaude, Aider, OpenHands, Goose, LangGraph, Pydantic AI, LlamaIndex, Haystack, Mem0, and Letta as upstream references or external tools, not as modules to reimplement.
- The goal is minimal custom work. If a capability can be provided by OpenCode configuration, a skill, an MCP server, or an upstream tool wrapper, use that path.
- Any new custom code must be a thin adapter with a clear reason: invoking an upstream tool, converting formats, running evaluations, or bridging a missing MCP endpoint.

## Agent Use

- Use `architect` for bottom-layer architecture decisions, framework selection, and capability mapping.
- Use `researcher` for read-only upstream comparison and current documentation checks.
- Use `critic` to review whether a proposed design is still too custom or too expensive to maintain.
- Use the built-in `build` agent only when the user asks for actual implementation.
- Use the built-in `plan` agent or `architect` for design-only work.

## Safety And Cost

- Keep secrets out of repo files. Provider keys and private base URLs belong in OpenCode global config, environment variables, or secure local files.
- Ask before file edits, shell commands with side effects, network-heavy MCPs, or external-directory access.
- Prefer disabled-by-default MCP examples in project config; enable only the server needed for the current task.

## Deliverable Rule

When making architecture changes, keep the result in docs and OpenCode-native config first. A good result should explain which OpenCode feature owns each capability and what, if anything, remains custom.

---
> Source: [redmaplewww/DeepReason-Agents-Framework](https://github.com/redmaplewww/DeepReason-Agents-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
