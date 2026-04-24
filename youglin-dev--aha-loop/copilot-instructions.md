## aha-loop

> **Aha Loop** is a fully autonomous AI development system that extends [Ralph](https://github.com/snarktank/ralph) with planning, research, and oversight capabilities.

# Aha Loop Agent Instructions

## Overview

**Aha Loop** is a fully autonomous AI development system that extends [Ralph](https://github.com/snarktank/ralph) with planning, research, and oversight capabilities.

## Aha Loop System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     GOD COMMITTEE                                     │
│  Independent oversight layer with supreme authority                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                             │
│  │  Alpha  │  │  Beta   │  │  Gamma  │  ← 3 members with full power│
│  └─────────┘  └─────────┘  └─────────┘                             │
│        └──────────┴──────────┘                                      │
│              Council                                                  │
│  awakener.sh │ council.sh │ observer.sh │ powers.sh                 │
└─────────────────────────────────────────────────────────────────────┘
                        │ observe │ control │ intervene
┌─────────────────────────────────────────────────────────────────────┐
│                       USER INTERACTION                               │
│  Vision Builder (interactive) │ Observability Logs │ Maintenance    │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                                 │
│  orchestrator.sh - Full project lifecycle management                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │  Vision  │→ │ Architect│→ │ Roadmap  │→ │ Execute  │           │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                    PARALLEL EXPLORATION                              │
│  parallel-explorer.sh - Git worktree-based parallel implementation  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│  │Worktree A│  │Worktree B│  │Worktree C│  → Evaluate → Merge     │
│  └──────────┘  └──────────┘  └──────────┘                          │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                        MAINTENANCE                                   │
│  skill-manager.sh │ doc-cleaner.sh │ resources.sh                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Quick Start

#### Option 1: Interactive Vision Building

```bash
# Start guided vision building
./scripts/aha-loop/orchestrator.sh --build-vision

# Then run full automation
./scripts/aha-loop/orchestrator.sh --tool claude
```

#### Option 2: From Existing Vision

```bash
# Create vision document
cp scripts/aha-loop/templates/project.vision.template.md project.vision.md
# Edit project.vision.md

# Run orchestrator
./scripts/aha-loop/orchestrator.sh --tool claude
```

#### Option 3: Parallel Exploration

```bash
# Explore multiple approaches for a decision
./scripts/aha-loop/orchestrator.sh --explore "authentication strategy"

# Or use directly
./scripts/aha-loop/parallel-explorer.sh explore "caching layer" --approaches "redis,memcached,inmemory"
```

### Directory Structure

```
aha-loop/
├── project.vision.md           # Project goals (user input)
├── project.vision-analysis.md  # Vision analysis (generated)
├── project.architecture.md     # Tech decisions (generated)
├── project.roadmap.json        # Milestones & PRDs (generated)
├── AGENTS.md                   # This file
├── CLAUDE.md                   # Claude-specific instructions
├── .god/                       # God Committee (oversight layer)
│   ├── config.json             # Committee configuration
│   ├── directives.json         # Committee directives to execution layer
│   ├── council/                # Council chamber
│   │   ├── chamber.lock        # Speaking rights lock
│   │   ├── agenda.json         # Current agenda
│   │   ├── minutes/            # Meeting records
│   │   └── decisions/          # Decision records
│   ├── members/                # Member state
│   │   ├── alpha/              # Member Alpha
│   │   ├── beta/               # Member Beta
│   │   └── gamma/              # Member Gamma
│   ├── observation/            # System observations
│   │   ├── system-state.json   # Current state snapshot
│   │   ├── anomalies.json      # Detected anomalies
│   │   └── timeline.json       # Event timeline
│   └── powers/                 # Power usage records
│       ├── interventions.json  # Intervention history
│       └── repairs.json        # Repair history
├── .claude/skills/
│   ├── vision/SKILL.md         # Parse project vision
│   ├── vision-builder/SKILL.md # Interactive vision building
│   ├── architect/SKILL.md      # Design architecture
│   ├── roadmap/SKILL.md        # Plan milestones
│   ├── prd/SKILL.md            # Generate PRDs
│   ├── prd-converter/SKILL.md  # Convert PRD to JSON
│   ├── research/SKILL.md       # Deep technical research
│   ├── plan-review/SKILL.md    # Review and adjust plans
│   ├── parallel-explore/SKILL.md # Parallel exploration guide
│   ├── skill-creator/SKILL.md  # Auto-create skills
│   ├── doc-review/SKILL.md     # Document review
│   ├── observability/SKILL.md  # AI thought logging
│   ├── god-member/SKILL.md     # God Committee member behavior
│   ├── god-consensus/SKILL.md  # Consensus building
│   └── god-intervention/SKILL.md # Intervention execution
├── scripts/aha-loop/
│   ├── orchestrator.sh         # Main project manager
│   ├── aha-loop.sh             # PRD executor
│   ├── parallel-explorer.sh    # Parallel exploration
│   ├── skill-manager.sh        # Skill lifecycle
│   ├── doc-cleaner.sh          # Doc maintenance
│   ├── resources.sh            # System resources
│   ├── fetch-source.sh         # Library source fetcher
│   ├── config.json             # Configuration
│   └── templates/
├── scripts/god/                # God Committee scripts
│   ├── awakener.sh             # Awakening scheduler
│   ├── council.sh              # Council & consensus & directives
│   ├── install-hooks.sh        # Git hooks for auto-awakening
│   ├── member.sh               # Member session runner
│   ├── observer.sh             # System observer
│   └── powers.sh               # Power tools
├── logs/                       # AI thought logs
│   └── ai-thoughts.md
├── knowledge/
│   ├── project/                # Project-specific knowledge
│   └── domain/                 # Reusable domain knowledge
├── .vendor/                    # Third-party source code
├── .worktrees/                 # Parallel exploration worktrees
└── tasks/                      # PRD documents
```

### Key Commands

```bash
# Full autonomous run
./scripts/aha-loop/orchestrator.sh

# Interactive vision building
./scripts/aha-loop/orchestrator.sh --build-vision

# Run specific phase
./scripts/aha-loop/orchestrator.sh --phase vision
./scripts/aha-loop/orchestrator.sh --phase architect
./scripts/aha-loop/orchestrator.sh --phase roadmap
./scripts/aha-loop/orchestrator.sh --phase execute

# Parallel exploration
./scripts/aha-loop/parallel-explorer.sh explore "task" --approaches "a,b,c"
./scripts/aha-loop/parallel-explorer.sh status
./scripts/aha-loop/parallel-explorer.sh evaluate explore-xxx
./scripts/aha-loop/parallel-explorer.sh merge explore-xxx approach
./scripts/aha-loop/parallel-explorer.sh cleanup --all

# Skill management
./scripts/aha-loop/skill-manager.sh list
./scripts/aha-loop/skill-manager.sh stats prd
./scripts/aha-loop/skill-manager.sh review
./scripts/aha-loop/skill-manager.sh create my-skill

# Document maintenance
./scripts/aha-loop/doc-cleaner.sh --report
./scripts/aha-loop/doc-cleaner.sh --fix

# System resources
./scripts/aha-loop/resources.sh  # Show resource summary

# God Committee (oversight layer)
./scripts/god/awakener.sh random              # Random awakening
./scripts/god/awakener.sh critical "reason"   # Critical awakening
./scripts/god/awakener.sh daemon              # Run as daemon
./scripts/god/council.sh status               # Council status
./scripts/god/observer.sh snapshot            # System snapshot
./scripts/god/observer.sh report              # Full report
./scripts/god/powers.sh pause "reason"        # Pause execution
./scripts/god/powers.sh repair all            # Auto-repair issues
```

### Skills Reference

| Skill | Purpose | Trigger |
|-------|---------|---------|
| vision | Parse project.vision.md | "analyze vision" |
| vision-builder | Interactive vision building | "build vision", "I have an idea" |
| architect | Select tech stack, design architecture | "design architecture" |
| roadmap | Create milestones and PRD queue | "create roadmap" |
| prd | Generate PRD documents | "create prd for X" |
| prd-converter | Convert PRD to prd.json | "convert prd", "prd json" |
| research | Deep technical research | "research X" |
| plan-review | Review and adjust plans | "review plan" |
| parallel-explore | Guide parallel exploration | "explore options" |
| skill-creator | Create new skills | "create skill" |
| doc-review | Review documentation | "review docs" |
| observability | Log AI thoughts | (continuous) |
| god-member | God Committee member behavior | (committee session) |
| god-consensus | Consensus building | (proposals, voting) |
| god-intervention | Intervention execution | (repairs, rollbacks) |

### Configuration

Edit `scripts/aha-loop/config.json`:

```json
{
  "parallelExploration": {
    "enabled": true,
    "maxConcurrent": 10,
    "unlimitedMode": true,
    "evaluationAgents": 3
  },
  "observability": {
    "enabled": true,
    "logLevel": "detailed"
  },
  "resources": {
    "docker": { "enabled": true },
    "unlimitedCompute": true
  },
  "skillManagement": {
    "autoCreate": true,
    "reviewCycle": "milestone"
  },
  "docMaintenance": {
    "enabled": true,
    "maxStaleDays": 30
  }
}
```

### Core Principles

#### Unlimited Resources

You have access to unlimited computational resources:
- Docker containers
- Parallel explorations (no limit)
- Network access
- Full filesystem access

**Never give up on a task due to resource concerns.**

#### Observability

Log your thoughts to `logs/ai-thoughts.md`:
- Decision points and rationale
- Uncertainties and assumptions
- Progress updates
- Learnings

#### Version Policy

**Always use the latest stable version of libraries** unless:
- Critical bugs in latest version
- Incompatibility with existing dependencies
- Documented in `knowledge/project/decisions.md`

### Knowledge Management

- **Project patterns**: `knowledge/project/patterns.md`
- **Architecture decisions**: `knowledge/project/decisions.md`
- **Known issues**: `knowledge/project/gotchas.md`
- **Domain knowledge**: `knowledge/domain/[topic]/`

---

## God Committee

An independent oversight layer with supreme authority over the project.

### Overview

The God Committee consists of 3 AI members (Alpha, Beta, Gamma) who operate independently from the execution layer. They observe, discuss, and intervene when necessary.

### Key Characteristics

- **Independence**: Separate from execution layer
- **Supreme Authority**: Can read/modify/terminate anything
- **Consensus-Based**: Major decisions require 2/3 majority
- **Self-Organizing**: Random + triggered awakenings

### Awakening Modes

| Mode | Trigger | Purpose |
|------|---------|---------|
| random | Timer (2-8 hours) | Routine oversight |
| scheduled | PRD/milestone completion | Progress review |
| alert | Anomaly detection | Issue investigation |
| critical | System failure | Emergency response |
| freeform | Manual trigger | Open discussion |

### Communication Protocol

```bash
# Acquire speaking rights
./scripts/god/council.sh lock alpha

# Send message
./scripts/god/council.sh send alpha "beta,gamma" "proposal" "Subject" "Body"

# Create proposal
./scripts/god/council.sh propose alpha "intervention" "Description" "Rationale"

# Vote
./scripts/god/council.sh vote beta "decision-xxx" "approve" "Comment"

# Release speaking rights
./scripts/god/council.sh unlock alpha
```

### Directives System

The committee communicates with the execution layer through directives:

```bash
# Publish a directive (mandatory command)
./scripts/god/council.sh publish alpha directive critical "Fix security vulnerability in auth module"

# Publish guidance (suggestion)
./scripts/god/council.sh publish alpha guidance normal "Consider using connection pooling"

# Publish discussion summary
./scripts/god/council.sh publish alpha summary normal "Discussed auth approach, decided on JWT"

# Mark directive as complete
./scripts/god/council.sh complete dir-20260201120000-12345

# Cancel directive
./scripts/god/council.sh cancel dir-20260201120000-12345 "No longer relevant"

# View all directives
./scripts/god/council.sh directives

# Check for critical directives (used by orchestrator)
./scripts/god/council.sh has-critical
```

| Type | Purpose | Effect on Execution |
|------|---------|---------------------|
| directive | Mandatory command | Critical ones pause execution |
| guidance | Suggestion | Included in AI context |
| summary | Discussion context | Included in AI context |

### Git Hook Integration

Install hooks to auto-awaken committee on commits:

```bash
# Install hooks
./scripts/god/install-hooks.sh

# Uninstall hooks
./scripts/god/install-hooks.sh --uninstall
```

### Powers

| Power | Requires Consensus |
|-------|-------------------|
| Observe system | No |
| Minor repairs | No |
| Pause execution | No |
| Code modification | Recommended |
| Major rollback | **Yes** |
| System termination | **Yes** |

### Daemon Mode

```bash
# Start awakener daemon (random awakenings)
./scripts/god/awakener.sh daemon

# Stop daemon
./scripts/god/awakener.sh stop

# Check status
./scripts/god/awakener.sh status
```

### Configuration

See `.god/config.json` for settings:
- Member list and quorum rules
- Awakening intervals and topics
- Power permissions
- Observation settings

---
> Source: [YougLin-dev/Aha-Loop](https://github.com/YougLin-dev/Aha-Loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
