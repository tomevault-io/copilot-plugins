## dcode-agent-kit

> This repo is the home of the **`/new-dcode-agent`** Claude Code skill, which scaffolds Deep Agents

# dcode-agent-kit, project context

This repo is the home of the **`/new-dcode-agent`** Claude Code skill, which scaffolds Deep Agents
(LangChain) agents and dcode CLI agents. A session opened here should act as a deepagents/dcode helper.

## Layout
- `skills/new-dcode-agent/SKILL.md`: the skill, the whole wizard, self-contained. It embeds the
  agent connector and templates and writes a self-contained agent into the user's current project.
- `.claude-plugin/{plugin.json, marketplace.json}`: let the skill install as a plugin via a marketplace.
- `reference/deepagents-guide.md`: the Deep Agents SDK + dcode CLI knowledge base.
- `assets/dak.png`: the banner.

## Conventions
- Provider-agnostic: scaffolded agents read their model and provider from env (`LLM_API_KEY`,
  optional `LLM_BASE_URL`, `LLM_MODEL`). Never bake a provider, model id, or key into generated code.
- A scaffolded SDK agent is a self-contained folder: `agent.py` + a sibling `model.py` (a
  same-directory import, no sys.path hacks). A mutating agent gates its tools with `interrupt_on`
  plus a checkpointer.
- Secrets come from env / `.env` (gitignored). Never commit keys.

## Verified essentials (deepagents 0.6.x)
- `create_deep_agent(model=..., tools=[...], system_prompt="...", ...)`. The prompt param is
  **`system_prompt`** (not `instructions`). `model` accepts `"provider:model"` or a `BaseChatModel` instance.
- `interrupt_on={...}` **requires** a `checkpointer` (e.g. `InMemorySaver`) or it no-ops.
- Verify SDK facts against the installed package before asserting them; the API moves.

## Editing the skill
- Keep the skill self-contained: the connector and the agent templates stay embedded in `SKILL.md`,
  so it works whether installed standalone (`~/.claude/skills/`) or as a plugin. No runtime
  dependency on this repo.
- Keep everything em-dash-free.

---
> Source: [EliaAlberti/dcode-agent-kit](https://github.com/EliaAlberti/dcode-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
