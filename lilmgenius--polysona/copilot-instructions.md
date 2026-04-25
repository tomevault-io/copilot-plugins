## polysona

> | Name | Role | Invocation Method | Key Capabilities |

# Polysona — Build and run multiple personas across any AI agent.

## Agent Catalog

| Name | Role | Invocation Method | Key Capabilities |
|---|---|---|---|
| profiler | Deep psychology interviewer | Codex: `$interview` / Claude Code: `/interview` | 10-framework interview flow, defense-bypass prompts, interview-log extraction |
| trendsetter | Trend detector | Codex: `$trend` / Claude Code: `/trend` | Domain trend scanning, topic prioritization, platform-fit filtering |
| content-writer | Platform content generator | Codex: `$content [platform]` / Claude Code: `/content [platform]` | Persona-conditioned drafting, platform pattern adaptation, CTA shaping |
| virtual-follower | QA simulator (`context:fork`) | Codex: `$qa` / Claude Code: `/qa` | Multi-audience simulation, rolemodel-gap scoring, TOP 5 draft recommendation |
| admin | Publisher and tracker | Codex: `$publish` / Claude Code: `/publish` | Publishing flow, scheduling, metadata capture, feedback loop updates |

## Architecture Overview

```text
┌─────────────────────────────────────────────────────────────┐
│                         POLYSONA                            │
│      Build and run multiple personas across any AI agent.   │
│                                                             │
│  SETUP (one-time / periodic refresh)                        │
│    ① profiler -> interview log extraction                   │
│           -> Polysona structuring engine                    │
│           -> persona.md + nuance.md + accounts.md           │
│                                                             │
│  LOOP (per content cycle)                                   │
│    ② trendsetter -> ③ content-writer -> ④ virtual-follower │
│    -> user selection -> ⑤ admin (publish + tracking)       │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start (Codex)

- `AGENTS.md` is auto-recognized by Codex-compatible workflows.
- Codex auto-discovers repo skills from `.agents/skills`. This repo mirrors `skills/` there for Codex compatibility.
- After editing `skills/`, resync with `node ./scripts/sync-codex-skills.mjs` or `bun run codex:skills:sync`.
- Use `$interview`, `$introduce`, `$trend`, `$content [platform]`, `$qa`, `$publish`, `$status`, `$export`.
- Keep persona data in `personas/{id}/` using PLOON format.

## Quick Start (Claude Code)

```bash
claude plugin marketplace add ./.claude-plugin/marketplace.json
claude plugin install polysona
```

- Use `/interview`, `/introduce`, `/trend`, `/content [platform]`, `/qa`, `/publish`, `/status`, `/export`.
- Keep lifecycle hooks in `hooks/hooks.json`. Do not assume every host auto-runs them the same way.

---
> Source: [LilMGenius/polysona](https://github.com/LilMGenius/polysona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
