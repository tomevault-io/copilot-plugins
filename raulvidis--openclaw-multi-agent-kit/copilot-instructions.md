## openclaw-multi-agent-kit

> > **RULE: Update this file whenever code or project structure changes.** Every added, removed, or renamed file, template, config key, or architectural decision must be reflected here before the task is considered done.

# CLAUDE.md — Project Rules & Context

> **RULE: Update this file whenever code or project structure changes.** Every added, removed, or renamed file, template, config key, or architectural decision must be reflected here before the task is considered done.

---

## Project Overview

**OpenClaw Multi-Agent Kit** — Production-tested templates for building AI agent teams on [OpenClaw](https://openclaw.sh) with Telegram supergroup integration. This is a template/docs-only repo (no runtime code). It provides SOUL.md personality templates, IDENTITY.md metadata templates, workspace scaffolding, and openclaw.json config snippets for deploying up to 10 autonomous agents coordinated through Telegram topic channels.

**Not a library or app.** Nothing here executes — it's all markdown templates, JSON config examples, and documentation meant to be copied into an OpenClaw workspace.

## Tech Stack / Platform

- **Platform:** OpenClaw (agent orchestration platform)
- **Channel:** Telegram (supergroups with topics)
- **LLM providers:** Anthropic Claude models (sonnet-4-6, opus-4-6, haiku-4-5)
- **Config format:** JSON / JSONC
- **Docs/templates:** Markdown

## Repository Structure

```
.
├── .gitignore                         # OS artifacts and editor files
├── CLAUDE.md                          # This file — project rules & context
├── INSTRUCTIONS.md                    # AI-readable setup guide (8 phases)
├── README.md                          # Project overview & quick start
├── LICENSE                            # MIT
├── docs/
│   ├── agent-design-patterns.md       # How to write effective SOUL.md files
│   ├── scaling.md                     # Scaling guidance: when to add agents, cost, circular triggers
│   ├── supergroup-setup.md            # Step-by-step Telegram supergroup setup (covers multi-bot + native topic routing)
│   ├── skills-system.md               # Native skills system, ClawHub, creating custom skills
│   ├── acpx-telegram.md               # ACPX coding agent backend for Telegram
│   └── telegram-dm-topics.md          # Telegram DM forum topics + ACP binding guide
├── examples/
│   ├── full-team.json                 # Complete 10-agent openclaw.json config
│   └── minimal-team.json              # Minimal 3-agent config (orchestrator + coder + QA)
└── templates/
    ├── openclaw-config.jsonc          # Base config snippet with defaults
    ├── identity/
    │   └── agent-identity.md          # Standard identity template for any agent
    ├── soul/                          # Agent personality templates (SOUL.md)
    │   ├── orchestrator.md            # Lead agent — coordinates all others
    │   ├── coding-agent.md            # Software engineering specialist
    │   ├── qa-agent.md                # Testing and quality assurance
    │   ├── devops-agent.md            # Infrastructure and deployment
    │   ├── research-agent.md          # Market research and intelligence
    │   ├── growth-agent.md            # Analytics and growth experiments
    │   ├── content-agent.md           # Social media content creation
    │   ├── community-agent.md         # Community engagement (Reddit, forums)
    │   ├── leadgen-agent.md           # Prospect research and lead scoring
    │   └── ops-agent.md              # Email, calendar, and data management
    ├── workspace/                     # Shared context file templates
    │   ├── AGENTS.md                  # Orchestrator operations guide
    │   ├── FEEDBACK-LOG.md            # Style corrections and lessons
    │   ├── SIGNALS.md                 # Shared intelligence hub
    │   ├── SUPERGROUP-MAP.md          # Topic and agent mapping
    │   └── THESIS.md                  # Business thesis — north star for all agents
    └── skills/                        # Skill templates (SKILL.md)
        ├── coding-handoff/            # Build→QA→Deploy handoff lifecycle
        ├── research-intel/            # Signal extraction + confidence scoring
        ├── leadgen-qualification/     # ICP scoring + outreach routing
        ├── content-repurpose/         # Cross-channel post repurposing
        ├── ops-triage/                # Priority routing for inbox/calendar/tasks
        ├── telegram-topic-setup/      # Automated topic creation and binding
        └── acpx-session/              # ACPX session management patterns
```

## Architecture — Key Concepts

- **Three Telegram routing models** — Multi-bot routing, native topic routing, and DM forum topics (see docs/)
- **One topic per team** — Teams share a topic channel in a supergroup
- **Primary + Secondary agents** — Primary owns the topic; secondary responds only when @mentioned or triggered
- **Shared context via markdown files** — Agents coordinate through THESIS.md, SIGNALS.md, FEEDBACK-LOG.md (not APIs)
- **Bot-to-bot via `sessions_send`** — Telegram bots cannot see each other's messages; OpenClaw's `sessions_send` bridges them
- **Structured escalation** — Agents escalate to orchestrator; orchestrator escalates to human

### Telegram Routing Models

| Model | Visibility | Best For |
|-------|-----------|----------|
| **Multi-bot routing** | Each agent has its own bot identity | Specialist teams with visible personas |
| **Native topic routing** | One bot, different internal agents per topic | Clean single-bot UX with internal specialization |
| **DM forum topics** | Topics inside a direct chat | Private 1:1 organized conversations with ACP support |

### Skills System (v2026.3.24+)

OpenClaw includes a native Skills system with one-click install from ClawHub, Control UI management, and CLI tools. Skills are placed at `agents/<agent>/skills/<skill-name>/SKILL.md` and use YAML frontmatter + markdown body format.

### ACPX Coding Subagents

Agents can delegate coding work to dedicated coding agents (Claude Code, Codex, OpenCode) via ACPX. Define a coder agent with `runtime.type: "acp"` in openclaw.json. Other agents trigger it via `sessions_spawn(runtime="acp")` from a subagent session. Thread bindings with `spawnAcpSessions` create persistent sessions per topic.

### Team Layout (default)

| Team     | Topic    | Primary Agent | Secondary Agents    |
|----------|----------|---------------|---------------------|
| General  | Topic 1  | Orchestrator  | —                   |
| Build    | Topic N  | Coder         | QA, DevOps          |
| Research | Topic N  | Researcher    | Growth              |
| Social   | Topic N  | Content       | Community           |
| Leads    | Topic N  | Lead Gen      | —                   |
| Ops      | Topic N  | Ops           | —                   |

The coder agent can optionally use ACPX (`runtime.type: "acp"`) to delegate to coding subagents. Add an ACP binding in the Build topic for persistent coding sessions.

### Critical Config Rule: `requireMention`

- **Multi-agent topics:** ALL bots must have `requireMention: true` — otherwise one bot responds to everything
- **Single-agent topics:** The sole agent can use `requireMention: false`
- **Orchestrator:** Must have `enabled: false` on topics owned by other agents

## Conventions

- **Template placeholders** use `[Your ... Name]`, `YOUR_*`, `[Name]`, or `{WORKSPACE}` — always replace before use
- **SOUL.md structure** follows the 10-section pattern from `docs/agent-design-patterns.md`: Identity, Who I Am, Core Principles, How I Work, Domain Sections, Communication Style, Shared Context, Team Integration, Learning/Memory, Success Metrics
- **Model selection:** Orchestrator/Coder use sonnet-4-6 or opus-4-6; lighter agents (QA, DevOps, Ops, Community) use haiku-4-5
- **Example configs** must stay in sync — `full-team.json` covers all 10 agents, `minimal-team.json` covers orchestrator + coder + QA
- **Skills format** uses YAML frontmatter (`name`, `description`, `version`) + markdown body with `## Install` section
- **ACPX agents** supported: claude, codex, opencode, gemini, pi, copilot, cursor, droid, kimi, kiro, qwen, trae
- **ACP config** goes at top level in openclaw.json with `acp.enabled`, `acp.backend`, `acp.allowedAgents`

## Editing Guidelines

- When adding a new agent template: add the soul template in `templates/soul/`, update `README.md` tables, update `examples/full-team.json`, and update this file's structure tree
- When adding a new workspace template: add in `templates/workspace/`, update `README.md`, and update this file
- When changing config schema or keys: update `templates/openclaw-config.jsonc`, both example files, and `INSTRUCTIONS.md`
- Keep `INSTRUCTIONS.md` as the single source of truth for the AI-readable setup flow
- All markdown templates use `---` horizontal rules as section separators

---
> Source: [raulvidis/openclaw-multi-agent-kit](https://github.com/raulvidis/openclaw-multi-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
