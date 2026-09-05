## pm-bot-sdlc-prompt

> Load when doing PM-Bot or SDLC workflow work


# PM-Bot SDLC & Workflow

## When to Load
- Planning or building SDLC skills (feature-planner, code-reviewer, log-analyzer, backtester)
- Working on PM-Bot Discord/OpenClaw integration
- Implementing Lobster pipelines (sdlc-full, rca-debug, tdlc-backtest)

## Key References
- **Roadmap**: `docs/planning/PM-BOT-ROADMAP-AND-NEXT-STEPS.md` — current status, exec capability, next steps
- **Vision**: `docs/vision/PM-BOT-VISION.md`
- **Community Skills**: `docs/vision/PM-BOT-COMMUNITY-SKILLS.md`
- **Exec troubleshooting**: `TOOLS.md` — if exec denied, `/reset` in Discord DM fixes it
- **AGENTS.md**: Skill invocation patterns, natural language matching

## Exec Capability
- **Direct exec works**: User can approve commands via Discord buttons
- **If exec denied**: Send `/reset` or `/new` in Discord DM — stale session cache causes deny; reset clears it
- **Evidence**: `EVIDENCE-exec-denied-vs-skill-creator.md`

## Skill Locations (workspace: /opt/pm-bot)
- SDLC: `skills/sdlc/feature-planner/`, `skills/sdlc/code-reviewer/`
- RCA: `skills/rca/log-analyzer/`
- TDLC: `skills/tdlc/backtester/`
- Flat symlinks in `skills/` for OpenClaw discovery (e.g. `feature-planner` → `sdlc/feature-planner`)

## Workflow
- Use **manual** approach for existing placeholder skills — flesh out Implementation in SKILL.md
- Use **skill-creator** for new workflow types (e.g. IT incident-responder)
- Pipelines: `lobster/pipelines/sdlc-full.lobster`, `rca-debug.lobster`, `tdlc-backtest.lobster`

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
