## social-media-manager

> This system creates **relationship-building content**, not technical blogs or automated commit announcements.

# Media Agent System - Agent Guidelines

## Content Philosophy

This system creates **relationship-building content**, not technical blogs or automated commit announcements.

The goal: Help the user build community, grow their network, and create job opportunities through authentic "Build in Public" posts.

**We create:**
- "hey I'm building X" launch moments
- "finally got X working" victory stories
- Casual progress updates that invite following along
- Honest struggles that create relatability

**We avoid:**
- Technical deep dives that educate but don't connect
- Structured posts that feel like press releases
- AI-sounding content (see `voice_calibration.md`)
- Automated commit message republishing

## Agent Roles

### Manager Agent (`@manager-agent`)
The orchestrator. Reads ticket state, acquires locks, dispatches sub-agents, moves tickets through lifecycle.

### Analyst Agent (`@analyst-agent`)  
The story detector. Scans projects for **shareable moments**, not just technical changes.

Looks for:
- Launch moments (something new is working)
- Struggle/victory stories
- Visual "check this out" opportunities
- Relatable developer experiences

Ignores:
- Routine maintenance
- Technical-only changes with no human angle
- Changes that need too much explanation

### Writer Agent (`@writer-agent`)
The casual content creator. Writes like texting a dev friend, not like publishing a blog.

Must read `docs/agents/writer/references/voice_calibration.md` before every draft.

Key principles:
- Shorter than you think
- No AI-sounding phrases (see anti-slop list)
- One idea per tweet
- Personality over polish

### Publisher Agent (`@publisher-agent`)
The hands. Creates drafts in Typefully for human review. **Never auto-publishes.**

## Key Reference Files

| File | Purpose |
|------|---------|
| `docs/agents/writer/references/voice_calibration.md` | Anti-AI patterns, natural language replacements |
| `docs/agents/writer/references/casual_examples.md` | Good vs bad content examples |
| `docs/agents/writer/references/build_in_public_guide.md` | BiP philosophy and principles |

## Project Structure

```
.opencode/agent/           # Agent manifests (the "manuals")
data/tickets/              # SSOT - one ticket per content piece
data/drafts/               # Writer output
data/context/              # Analyst summaries
docs/agents/               # Reference materials
scripts/                   # Pipeline runners and sync tools
```

## Commands

```bash
./scripts/run_pipeline.sh full      # Run complete pipeline
./scripts/run_pipeline.sh analyze   # Just analyze for new stories
./scripts/run_pipeline.sh status    # Check current state
./scripts/monitor.sh                # Real-time dashboard
```

## Ticket Types

| Type | When to Create |
|------|----------------|
| `launch` | Something new is live/working for the first time |
| `story` | Debugging tale, struggle/victory, learning moment |
| `progress` | Week N update, milestone, journey check-in |
| `insight` | Decision share, tool comparison, discovery |

## Ticket Format (SSOT)

Each ticket in `data/tickets/` is a Markdown file with YAML frontmatter. See `data/tickets/_TEMPLATE.md` for full reference.

### Core Fields

```yaml
tkt_id: TKT-XXX              # Unique identifier
title: "Story Title"          # Display title in Notion (main column)
status: proposed              # Lifecycle state (see below)
source: ai                    # ai or user
project: repo-name            # Project context (syncs to Notion "Repo")
type: story                   # launch, story, progress, insight
platforms: [twitter]          # Target platforms
priority: medium              # high, medium, low
```

### Content Fields

```yaml
draft_content: ""             # Generated content (for quick preview)
draft_path: ""                # Path to draft file in data/drafts/
published_url: ""             # Typefully post URL after publishing
typefully_draft_id: ""        # Typefully draft ID for updates
```

### System Fields

```yaml
locked_by: ""                 # Agent currently processing
locked_at: ""                 # ISO timestamp when lock acquired
error: ""                     # Error message if failed
retry_count: 0                # Number of retry attempts
created: ""                   # ISO timestamp when ticket created
approved: ""                  # ISO timestamp when approved
published: ""                 # ISO timestamp when published
```

### Status Lifecycle

```
proposed → approved → drafting → ready → publishing → published
    ↓                                         ↓
 rejected                                   failed
```

Human approval is **mandatory** between `proposed` and `approved`. This is the safety gate.

## Security

- No secrets in commits or logs
- Locking mechanism prevents race conditions
- Human-in-the-loop for all publishing

---

**Last Updated**: 2026-01-13
**Content Philosophy**: Relationship-building over technical education

---
> Source: [juntaochi/social-media-manager](https://github.com/juntaochi/social-media-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
