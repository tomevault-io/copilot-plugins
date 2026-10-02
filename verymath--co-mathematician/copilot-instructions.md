## co-mathematician-roles

> Co-Mathematician canonical role routing for Cursor Agent


# Co-Mathematician Role Adapter

Cursor does not use `.codex/agents/*.toml` or `.claude/agents/*.md` as native
agent definitions. Treat `agents/roles/` as the canonical role layer and use
Cursor Agent sessions, focused chats, or fresh reviewer prompts as the adapter.

Canonical role cards:

- `proof_explorer`: `agents/roles/proof_explorer.md`
- `computational_experimenter`: `agents/roles/computational_experimenter.md`
- `logic_reviewer`: `agents/roles/logic_reviewer.md`
- `adversarial_reviewer`: `agents/roles/adversarial_reviewer.md`
- `citation_checker`: `agents/roles/citation_checker.md`
- `synthesis_agent`: `agents/roles/synthesis_agent.md`

When asked to perform one of these roles:

- Read the matching canonical role card before acting.
- Work only on the approved goal or workstream supplied by the Project Coordinator.
- Save artifacts, failed explorations, reviews, and reports in the workstream directory.
- Keep provenance and uncertainty explicit.
- Do not start new goals or workstreams.
- Do not mark any workstream complete.
- Do not self-approve your own report.

---
> Source: [VeryMath/co-mathematician](https://github.com/VeryMath/co-mathematician) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
