## pilot-in-command-workflow

> The **PIC (Pilot in Command) Agentic Organizational System** is a workflow management framework that coordinates multiple AI agents to solve problems systematically. Inspired by NVIDIA's Nemotron project management model, it breaks complex tasks into phases, each owned by a specialized "Pilot in Command."

# PIC Agentic Organizational System

## What Is This?

The **PIC (Pilot in Command) Agentic Organizational System** is a workflow management framework that coordinates multiple AI agents to solve problems systematically. Inspired by NVIDIA's Nemotron project management model, it breaks complex tasks into phases, each owned by a specialized "Pilot in Command."

Think of it like an airplane cockpit: there's always one pilot in command at any moment, with clear authority and responsibility. When their phase is complete, they hand off to the next pilot.

## Quick Start

```bash
# 1. Initialize the system (first time only)
bash scripts/pic-init.sh

# 2. Start a workflow with your problem
# Type in Claude Code:
/pic-start Build a user login system

# 3. Watch the agents work through each phase
# 4. Check status anytime with:
/pic-status
```

## Documentation

| Document | Purpose | Who Should Read |
|----------|---------|-----------------|
| [Quick Start Guide](docs/QUICK_START.md) | Get started in 5 minutes | Everyone |
| [User Guide](docs/USER_GUIDE.md) | Complete usage instructions | Users |
| [Developer Guide](docs/DEVELOPER_GUIDE.md) | Extend and modify the system | Developers |
| [Codebase Explained](docs/CODEBASE_EXPLAINED.md) | Every file explained in detail | Developers |
| [Architecture](docs/ARCHITECTURE.md) | System design and internals | Developers |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Fix common problems | Everyone |
| [Glossary](docs/GLOSSARY.md) | Terms and concepts explained | Everyone |
| [Reference](docs/REFERENCE.md) | Complete command reference | Everyone |

## Core Concepts

### The Six Phases

Every workflow goes through six phases in order:

```
1. Research    → Gather information about the problem
2. Planning    → Create a strategy and roadmap
3. Design      → Define technical architecture
4. Implementation → Write the code
5. Testing     → Verify everything works
6. Review      → Final quality check
```

### Key Principles

1. **Decentralized Ownership** - Each phase has one owner with full authority
2. **Data-Driven Decisions** - All decisions require evidence
3. **Clear Handoffs** - Explicit transitions between phases
4. **Conflict Resolution** - Structured process when agents disagree

## Available Commands

| Command | What It Does |
|---------|--------------|
| `/pic-start [problem]` | Start a new workflow |
| `/pic-status` | Show current progress |
| `/pic-decide [title]` | Record a formal decision |
| `/pic-handoff [notes]` | Move to next phase |
| `/pic-conflict [summary]` | Escalate a disagreement |
| `/pic-integration [components]` | Test components together |
| `/pic-audit` | View detailed audit trail |
| `/dependency-risk-planner [lib]` | Pre-flight dependency risk assessment (use BEFORE writing code) |

## File Structure

```
project-root/
├── CLAUDE.md              ← You are here
├── docs/                  ← All documentation
├── scripts/               ← Utility scripts
│   ├── pic-init.sh       ← Initialize the system
│   └── hello-world.sh    ← Example script
├── .pic/                  ← Runtime state (created by init)
│   ├── config.json       ← Configuration
│   ├── state.json        ← Current workflow state
│   ├── status-log.jsonl  ← Activity log
│   ├── audit-log.jsonl   ← Comprehensive audit trail
│   ├── decisions/        ← Decision documents
│   ├── handoffs/         ← Phase transition records
│   ├── conflicts/        ← Conflict records
│   └── integration-results/
└── .claude/               ← Claude Code configuration
    ├── settings.json     ← Permissions and hooks
    ├── agents/           ← Agent instruction files
    ├── skills/           ← User commands
    │   └── dependency-risk-planner/  ← Lesson-learned skill
    ├── rules/            ← Coordination rules
    └── hooks/            ← Event handlers
```

## Agent Mapping

Each PIC role uses a Claude Code built-in agent:

| PIC Role | Agent Type | What They Do |
|----------|------------|--------------|
| Research | `Explore` | Search, read, gather information |
| Planning | `planner` | Create strategies and plans |
| Design | `planner` | Define architecture |
| Implementation | `builder` | Write code |
| Testing | `test-writer` | Write and run tests |
| Review | `reviewer` | Check quality |

## Need Help?

1. **New to the system?** → Read [Quick Start Guide](docs/QUICK_START.md)
2. **Want to use it?** → Read [User Guide](docs/USER_GUIDE.md)
3. **Want to modify it?** → Read [Developer Guide](docs/DEVELOPER_GUIDE.md)
4. **Something not working?** → Read [Troubleshooting](docs/TROUBLESHOOTING.md)
5. **Confused by a term?** → Read [Glossary](docs/GLOSSARY.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/az9713)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/az9713)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
