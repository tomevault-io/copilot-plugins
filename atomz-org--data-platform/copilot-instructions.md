## data-platform

> Data platform protocol: where the rules are and which scope you are in

# Data platform — read these, in order

1. `CLAUDE.md` — the router: the directories, what is shared, what is read-only.
2. `AGENTS.md` — the protocol. §0 names your scope; the rest is per scope.
3. `.memory/MEMORY.md` — what earlier agents learned. `uv run pf memory show` from
   where you are working prints the notes that apply there.

You are the **Session** scope: a person is in the loop and you have a shell.
Ask the graph before reading files — `kg_search`, `kg_neighbors`, `impact_analysis`
are MCP tools from `.cursor/mcp.json`. Stay inside one project; never read a sister.

`.cursor/hooks.json` runs the same gate Claude Code runs, before every write, delete,
shell command and read (`docs/HARNESSES.md`). A refusal is a finding: report it; never
retry it another way. Leave a note before you finish (`AGENTS.md` §5).

---
> Source: [Atomz-org/data-platform](https://github.com/Atomz-org/data-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
