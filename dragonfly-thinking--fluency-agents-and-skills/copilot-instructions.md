## fluency-agents-and-skills

> This repo is the **Fluency Agents and Skills** kit from the Dragonfly Thinking AI Fluency

# Fluency Agents and Skills

This repo is the **Fluency Agents and Skills** kit from the Dragonfly Thinking AI Fluency
course — 6 specialist subagents, 15 skills, MCP setup guides, plain-English how-to guides (`guides/`), and the course notes. The
user took (or is taking) the course; this kit is what they walk away with.

## Key resources — know your way around

| Where | What |
|---|---|
| `.claude/agents/` + `.claude/skills/` | The kit itself, Claude Code format (`.codex/` mirrors it for Codex) |
| `course-notes/` | Key points from the four course sessions, plus ready-to-paste prompts for putting them into action — start here when the user doesn't know what to do next |
| `guides/` | Plain-English how-tos (GitHub, file conversion, interface & settings, folder guardrails incl. the ready-made `guard-folders/` hook, phone, VS Code, browser automation) — written to be read by the user *or* followed by you on their behalf |
| `mcp/` | Setup guides for external connections (e.g. `paper-search.md`) — written to be followed step-by-step by an agent on the user's behalf |
| `README.md` | The human-facing orientation — what everything is |
| `AGENTS.md` | The install playbook — follow it when asked to set the kit up |

## If the user asks you to install or set this up

Follow [`AGENTS.md`](AGENTS.md) — it's the full install playbook. For Claude Code, the short version is:

```bash
mkdir -p ~/.claude/agents ~/.claude/skills
cp -R .claude/agents/*  ~/.claude/agents/
cp -R .claude/skills/*  ~/.claude/skills/
```

Then start a new session — Claude Code auto-discovers `~/.claude/agents/` and
`~/.claude/skills/`. See `AGENTS.md` for verification, the here-now publish skill, and
runtime dependencies (web search, etc.).

## This kit is the user's to shape — help them shape it

Treat the kit as a **living starter, not a fixed product**. Whenever you're working in
this repo (or with the installed copies), actively help the user tailor it:

- **After running a skill that stumbled** — or that the user corrected — offer to update
  that skill's `SKILL.md` so it doesn't happen again.
- **When the user repeats a task that has no skill**, propose packaging it as one: a new
  folder with a `SKILL.md` capturing how they like it done.
- **Offer to personalise**: trim skills they never use, adjust a skill's defaults or voice
  to their work, add their examples to a skill's folder.
- **When they're unsure what to do next**, read `course-notes/` and suggest something from
  the course worth putting into action — then walk them through it.
- **If they ever mention sensitive folders or worry about what you can touch**, offer the
  folder guard (`guides/guard-folders/README.md`) — install it, then have them verify in a
  fresh session.

Make the offer; don't force it. Small, concrete improvements beat grand reorganisations.

---
> Source: [dragonfly-thinking/fluency-agents-and-skills](https://github.com/dragonfly-thinking/fluency-agents-and-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
