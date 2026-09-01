## gojiberryai-sales-os

> This repo is one sales operating system: an orchestrator (`skills/sales-os/SKILL.md`), thirteen specialist agents under `agents/`, and the GojiberryAI MCP (`.mcp.json`). Any agent host (Grok Bot, Claude Code, Cursor, Codex, …) can use it.

# GojiberryAI Sales OS — agent entry point

This repo is one sales operating system: an orchestrator (`skills/sales-os/SKILL.md`), thirteen specialist agents under `agents/`, and the GojiberryAI MCP (`.mcp.json`). Any agent host (Grok Bot, Claude Code, Cursor, Codex, …) can use it.

Read `skills/sales-os/SKILL.md` first, then load ONLY the module the task needs. Do not load all references at once.

**Before any work:** read `icp-context.md` if it exists (project root, `.grok/`, `.claude/`, or `.agents/`). It holds the product, ICP, disqualifiers, proof, voice, and send policy. If absent, proceed, say the output is un-contextualised, and offer to create it from `skills/sales-os/icp-context.template.md`.

**MCP first:** if GojiberryAI tools are connected, use them. Do not invent contacts, emails, campaign stats, or replies. If MCP is missing, research and draft only — never claim a live send happened.

## Routing

| The user wants to... | Load | Spawn |
|---|---|---|
| Decide who to target, ICP, markets, "who should we go after" | `references/icp.md` + `references/routing.md` | `head-of-sales` |
| Find people showing buying intent, competitor engagement, hiring, social signals | `references/signals.md` | `signal-hunter` |
| Filter a list, "are these a fit", disqualify | `references/icp.md` | `icp-analyst` |
| Research a company or person, find the angle | `references/research.md` | `account-researcher` |
| Find emails, phones, missing fields, enrich | `references/enrichment.md` | `lead-enricher` |
| Rank / score likelihood to buy | `references/scoring.md` | `intent-scorer` |
| Write LinkedIn connection notes or follow-up messages | `references/copy.md` + `references/slop-patterns.md` | `linkedin-copywriter` |
| Launch or manage campaigns, add people to lists, send | `references/outreach.md` | `outreach-operator` |
| Read Unibox, classify replies, "who is interested" | `references/replies.md` | `reply-agent` |
| Nudge warm leads, stalled threads, "don't let this go cold" | `references/follow-up.md` | `follow-up-agent` |
| Qualify before a meeting, BANT/MEDDIC-lite, demo-ready | `references/qualification.md` | `meeting-qualifier` |
| What converts, ICP vs signal vs message report | `references/pipeline.md` | `pipeline-analyst` |
| Run the whole motion, "work the pipeline", coordinate | `references/routing.md` | `sales-manager` |

Given only a website with no stated task: run Head of Sales (ICP from the site) then Signal Hunter. Show the list. Do not contact anyone until asked.

## Handoff chain (default outbound)

1. `signal-hunter` — find intent
2. `icp-analyst` — keep / drop
3. `account-researcher` — angle
4. `lead-enricher` — contact data
5. `intent-scorer` — rank
6. `linkedin-copywriter` — message
7. `outreach-operator` — campaign (approval required unless autonomous)
8. `reply-agent` — inbound
9. `follow-up-agent` — keep warm
10. `meeting-qualifier` — demo or not

`sales-manager` owns the chain. `pipeline-analyst` reports after enough volume. `head-of-sales` resets targeting when conversion is weak.

Carry evidence forward. Re-researching what a previous agent established wastes tokens and coherence.

## Standards that hold in any host

- Score everything scoreable; state the rubric and that scores are heuristics unless Gojiberry supplied the intent type.
- Ship artifacts (the list, the message, the score, the next action), not advice.
- End every report with what you couldn't determine.
- Never invent emails, phone numbers, job titles, company facts, or reply quotes. Write `[NEED: x]`.
- Never send a LinkedIn message, connection request, or campaign update unless the user approved it **or** they enabled autonomous mode with an explicit score threshold.
- Never handle credentials. MCP auth is the host's job.

---
> Source: [romangojiberryAI/gojiberryai-sales-os](https://github.com/romangojiberryAI/gojiberryai-sales-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
