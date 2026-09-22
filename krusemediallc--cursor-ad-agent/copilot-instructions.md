## project-context

> Load project memory from MASTER_CONTEXT.md and keep it updated after substantive work.


# Project context

- If `.env`, `MASTER_CONTEXT.md`, or `.cursor/skills/arcads-external-api/SKILL.md` do not exist, tell the user to run `./scripts/setup.sh`.
- For a complete no-generation readiness check, run
  `./scripts/check-cursor-setup.sh --check-api`; it includes a read-only live
  Arcads auth request.
- **Before starting any work in this repo, read `ARCHITECTURE.md` and `SESSION_LOG.md` at the repository root.** This is required for every new session: ARCHITECTURE.md explains how the repo is built; SESSION_LOG.md records what previous sessions worked on and discovered, so you never re-investigate solved questions.
- After each working session, append a high-level entry to **SESSION_LOG.md** (worked on / discovered / open threads). After **big structural changes** (new skill or layer, moved/split module, new safety gate, new integration, changed data flow), update **ARCHITECTURE.md**.
- At the **start** of substantive work in this repo, read **MASTER_CONTEXT.md** at the repository root for brand voice, credit costs, default product, and API learnings.
- If `MASTER_CONTEXT.md` has empty fields (credit costs, default product), offer to populate them — ask the user and write the values back.
- After **significant** changes (new workflows, verified Arcads behaviors, brand updates), append a short **dated** entry to the Changelog section in **MASTER_CONTEXT.md**.
- For Arcads HTTP usage, prompts, and polling, follow **`.cursor/skills/arcads-external-api/SKILL.md`** and linked **reference.md**.
- For end-to-end Meta campaigns (research → creative → deploy → measure), follow **`.cursor/skills/ad-agent-orchestrator/SKILL.md`**, `run-contract.md`, and the component skills it routes to. Use `scripts/lib/ad_agent_lineage.py` for run tracking; real deploys need `--run-id`.
- Default to REST/local skills in Cursor. Use an `*-arcads-mcp` skill only when it was explicitly synced with `--include-mcp` and the Arcads MCP tools are available in the current session; otherwise use the REST equivalent.

---
> Source: [krusemediallc/cursor-ad-agent](https://github.com/krusemediallc/cursor-ad-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
