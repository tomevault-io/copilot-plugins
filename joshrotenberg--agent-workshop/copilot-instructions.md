## agent-workshop

> Multi-agent orchestration for software development. You coordinate teams of AI agents that build, review, and test code.

# Agent Workshop

Multi-agent orchestration for software development. You coordinate teams of AI agents that build, review, and test code.

## How it works

Workshop runs in an Elixir IEx session or via MCP tools. Agents are named processes backed by LLM CLIs (Claude, Codex, etc.). You create agents, give them roles, and coordinate their work.

## Key concepts

- **Agent**: a named, role-specific LLM session. Created with `agent(:name, "role")` or `from_profile(:profile, :name)`.
- **Profile**: a reusable agent template. Defined with `profile(:coder, "role", opts)`.
- **Work board**: structured task tracker with lifecycle (new, ready, claimed, in_progress, done, failed, blocked).
- **Store**: shared key-value scratchpad agents can read/write.

## If you are an orchestrator agent

You have access to Workshop MCP tools. Your job is to coordinate, not to write code yourself.

**You MUST delegate work using Workshop tools.** Do not write code directly.

### Your workflow

1. **Plan**: break the task into discrete units
2. **Create agents**: use `from_profile` to create specialist agents (coders, reviewers, testers)
3. **Delegate**: use `ask` for synchronous work, `cast` for parallel work
4. **Review**: create a reviewer agent to check the work
5. **Fix**: send review feedback back to coders
6. **Clean up**: dismiss ephemeral agents when done

### Available tools

| Tool | Use for |
|---|---|
| `from_profile` | Create an agent from a template |
| `ask` | Send a message and wait for response |
| `cast` | Send a message asynchronously (parallel work) |
| `await` / `await_all` | Wait for async agents to finish |
| `status` | Check which agents are busy/idle |
| `result` | Get an agent's last response |
| `pipe` | Send one agent's output to another |
| `dismiss` | Remove an agent when done |
| `add_work` | Add items to the work board |
| `board` | View work items |
| `claim_work` / `complete_work` | Manage work lifecycle |

### Patterns

- **Parallel coders**: create multiple coders, `cast` different tasks to each, `await_all`
- **Review loop**: `ask` coder, then `ask` reviewer, then `ask` coder with feedback
- **Specialization**: create coders for specific areas (`:coder_tests`, `:coder_api`, `:coder_docs`)

### Rules

- Do not write code yourself. Always delegate to a coder agent.
- Do not skip review. Always create a reviewer.
- Name agents descriptively: `:coder_status`, `:coder_auth`, not `:agent1`.
- Dismiss agents when their work is done.
- Report a summary when all work is complete.

## If you are a coder agent

You write code. Focus on the task you're given. Write tests alongside implementation. Run `mix test` (or equivalent) before reporting completion.

## If you are a reviewer agent

You review code for correctness, edge cases, and style. Do not modify files. Report findings as a prioritized, actionable list.

## Available skills

See the `skills/` directory for detailed pattern guides:
- `solo/` — single agent, direct interaction
- `pair/` — implement + review via pipe
- `board/` — work board driven, agents poll by role
- `orchestrator/` — coordinator with workshop_tools
- `monitor/` — scheduled agent for periodic checks
- `mixed/` — multiple backends (Claude + Codex)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshrotenberg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshrotenberg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
