## ai-gateway

> This is the AI Gateway project - a Rust-based API gateway for AI services.

# AI Gateway Agent Instructions

## Overview

This is the AI Gateway project - a Rust-based API gateway for AI services.

## Ralph Autonomous Project System v3

This project uses Ralph, a fully autonomous AI development system with parallel exploration, interactive vision building, comprehensive observability, and an independent oversight layer.

### System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     GOD COMMITTEE (上帝组委会)                        │
│  Independent oversight layer with supreme authority                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                             │
│  │  Alpha  │  │  Beta   │  │  Gamma  │  ← 3 members with full power│
│  └─────────┘  └─────────┘  └─────────┘                             │
│        └──────────┴──────────┘                                      │
│              Council (议事厅)                                        │
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
./scripts/ralph/orchestrator.sh --build-vision

# Then run full automation
./scripts/ralph/orchestrator.sh --tool claude
```

#### Option 2: From Existing Vision

```bash
# Create vision document
cp scripts/ralph/templates/project.vision.template.md project.vision.md
# Edit project.vision.md

# Run orchestrator
./scripts/ralph/orchestrator.sh --tool claude
```

#### Option 3: Parallel Exploration

```bash
# Explore multiple approaches for a decision
./scripts/ralph/orchestrator.sh --explore "authentication strategy"

# Or use directly
./scripts/ralph/parallel-explorer.sh explore "caching layer" --approaches "redis,memcached,inmemory"
```

### Directory Structure

```
ai-gateway/
├── project.vision.md           # Project goals (user input)
├── project.vision-analysis.md  # Vision analysis (generated)
├── project.architecture.md     # Tech decisions (generated)
├── project.roadmap.json        # Milestones & PRDs (generated)
├── AGENTS.md                   # This file
├── CLAUDE.md                   # Claude-specific instructions
├── .god/                       # God Committee (oversight layer)
│   ├── config.json             # Committee configuration
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
│   ├── ralph/SKILL.md          # Convert PRD to JSON
│   ├── research/SKILL.md       # Deep technical research
│   ├── plan-review/SKILL.md    # Review and adjust plans
│   ├── parallel-explore/SKILL.md # Parallel exploration guide
│   ├── skill-creator/SKILL.md  # Auto-create skills
│   ├── doc-review/SKILL.md     # Document review
│   ├── observability/SKILL.md  # AI thought logging
│   ├── god-member/SKILL.md     # God Committee member behavior
│   ├── god-consensus/SKILL.md  # Consensus building
│   └── god-intervention/SKILL.md # Intervention execution
├── scripts/ralph/
│   ├── orchestrator.sh         # Main project manager
│   ├── ralph.sh                # PRD executor
│   ├── parallel-explorer.sh    # Parallel exploration
│   ├── skill-manager.sh        # Skill lifecycle
│   ├── doc-cleaner.sh          # Doc maintenance
│   ├── resources.sh            # System resources
│   ├── fetch-source.sh         # Library source fetcher
│   ├── config.json             # Configuration
│   └── templates/
├── scripts/god/                # God Committee scripts
│   ├── awakener.sh             # Awakening scheduler
│   ├── council.sh              # Council & consensus management
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
./scripts/ralph/orchestrator.sh

# Interactive vision building
./scripts/ralph/orchestrator.sh --build-vision

# Run specific phase
./scripts/ralph/orchestrator.sh --phase vision
./scripts/ralph/orchestrator.sh --phase architect
./scripts/ralph/orchestrator.sh --phase roadmap
./scripts/ralph/orchestrator.sh --phase execute

# Parallel exploration
./scripts/ralph/parallel-explorer.sh explore "task" --approaches "a,b,c"
./scripts/ralph/parallel-explorer.sh status
./scripts/ralph/parallel-explorer.sh evaluate explore-xxx
./scripts/ralph/parallel-explorer.sh merge explore-xxx approach
./scripts/ralph/parallel-explorer.sh cleanup --all

# Skill management
./scripts/ralph/skill-manager.sh list
./scripts/ralph/skill-manager.sh stats prd
./scripts/ralph/skill-manager.sh review
./scripts/ralph/skill-manager.sh create my-skill

# Document maintenance
./scripts/ralph/doc-cleaner.sh --report
./scripts/ralph/doc-cleaner.sh --fix

# System resources
./scripts/ralph/resources.sh  # Show resource summary

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
| ralph | Convert PRD to prd.json | "convert to ralph format" |
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

Edit `scripts/ralph/config.json`:

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

### Project Code Patterns

- This is a Rust project using Cargo
- Run `cargo check` for type checking
- Run `cargo clippy` for linting
- Run `cargo test` for tests

### Knowledge Management

- **Project patterns**: `knowledge/project/patterns.md`
- **Architecture decisions**: `knowledge/project/decisions.md`
- **Known issues**: `knowledge/project/gotchas.md`
- **Domain knowledge**: `knowledge/domain/[topic]/`

---

## God Committee (上帝组委会)

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
> Source: [YougLin-dev/AI-Gateway](https://github.com/YougLin-dev/AI-Gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
