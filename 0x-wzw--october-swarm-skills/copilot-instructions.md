## october-swarm-skills

> > **Hermes → Hermes equivalents** (all paths and commands below work on both platforms)

# SKILL.md - October Swarm Architecture

## PLATFORM_MAPPING

> **Hermes → Hermes equivalents** (all paths and commands below work on both platforms)

| Hermes Path / Command | Hermes Equivalent |
|-------------------------|-------------------|
| `~/.openclaw/workspace` | `~/.hermes/skills/autonomous-ai-agents/hermes-agent` |
| `~/.hermes/projects/october-swarm-skills/agents/octoberxin` | `~/.hermes/projects/october-swarm-skills/agents/octoberxin` |
| `~/.hermes/projects/october-swarm-skills/agents/halloween` | `~/.hermes/projects/october-swarm-skills/agents/halloween` |
| `~/.hermes/projects/october-swarm-skills/agents/octavia` | `~/.hermes/projects/october-swarm-skills/agents/octavia` |
| `~/.hermes/projects/october-swarm-skills/agents/octane` | `~/.hermes/projects/october-swarm-skills/agents/octane` |
| `~/.hermes/projects/october-swarm-skills/agents/octopus` | `~/.hermes/projects/october-swarm-skills/agents/octopus` |
| `~/.hermes/projects/october-swarm-skills/agents/bee` | `~/.hermes/projects/october-swarm-skills/agents/bee` |
| `~/.hermes/vault/credentials.json` | `~/.hermes/vault/credentials.json` |
| `~/.hermes/memory/daily/YYYY-MM-DD.md` | `~/.hermes/memory/daily/YYYY-MM-DD.md` |
| `~/.hermes/memory/user/MEMORY.md` | `~/.hermes/memory/user/MEMORY.md` |
| `~/.hermes/logs/agent-comm/` | `~/.hermes/logs/agent-comm/` |
| `~/obsidian-vault/` | `~/obsidian-vault/` |
| `hermes chat` | `hermes chat` |
| `openclaw october spawn <agent>` | `delegate_task` with context + skill loading |
| `openclaw october task init` | `/todo` or `todo` tool |
| `openclaw october workflow <stage>` | `delegate_task` with goal + toolsets |
| `cronjob or delegate_task for background tasks` | `cronjob` or `delegate_task` for background tasks |
| `delegate_task with classification goal` | Manual task classification (or `delegate_task` with classification goal) |
| `openclaw octorch route` | `delegate_task` with per-task model/provider |
| `openclaw octorch workflow start` | `delegate_task` batch with workflow stages |
| `openclaw octorch critique` | `delegate_task` with critique skill + adversarial-stress-test |
| `openclaw octorch topology` | `skills_list` + `skill_view` |
| `openclaw sprint start` | `todo` tool with sprint items |
| `openclaw bee task submit` | `cronjob` create for background tasks |
| `openclaw skill <name>` | `skill_view(name)` + follow skill instructions |

---
**October Swarm** — A multi-agent orchestration framework with tiered architecture, 4-stage workflows, and bee swarm administrative capabilities.

## Overview

October is a production-ready multi-agent orchestration framework designed for complex, autonomous task execution. It implements a tiered model (T1-T4) with specialized agents that collaborate through defined workflows.

**Key Features:**
- Tiered collaboration (T1-T4 models based on task complexity)
- 4-stage workflow: Planning → Review → QA → Ship
- Bee swarm administrative pool for stateless workers
- Agent topology and handoff protocols
- Sprint integration for time-boxed work cycles

**Version:** 1.0.0
**Status:** Alpha
**Channel:** Hermes / OpenClaw

---

## Installation

### Prerequisites

```bash
# Python 3.10+ or Node.js 18+
# ollama or LLM service configured
# Hermes installed and running
```

### Install the October Swarm

```bash
# Install via ClawHub
clawhub install october-swarm-skills

# Or clone manually
git clone <repository-url> ~/.hermes/skills/autonomous-ai-agents/hermes-agent
cd ~/.hermes/skills/autonomous-ai-agents/hermes-agent
```

### Install Individual Skills

```bash
clawhub install october-swarm-skills/swarm-orchestration
clawhub install october-swarm-skills/4-stage-workflow
clawhub install october-swarm-skills/bee-tasks
clawhub install october-swarm-skills/octoberxin-critique
clawhub install october-swarm-skills/agent-topology
clawhub install october-swarm-skills/sprint-cycles
```

### Setup Agent Workspaces

```bash
# Create agent directories
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/halloween
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octoberxin
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octavia
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octane
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octopus
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/bee/{pending,completed}

# Copy agent profiles
cp -r october-swarm-skills/agents/* ~/.hermes/projects/october-swarm-skills/agents/bee/
```

### Configure Credentials

```bash
cp examples/credentials.example ~/.hermes/vault/credentials.json
# Edit with actual credentials (Moltbook, Notion, X/GitHub, etc.)
```

---

## Quick Start

### Initialize October

```bash
# Start the orchestrator
hermes chat

# Check swarm status
hermes status

# Spawn an agent
delegate_task with context + skill loading for halloween
```

### Run a 4-Stage Workflow

```bash
# Create a new task
todo add "feature-auth"

# Execute workflow stages
delegate_task with goal + toolsets for planning  # Halloween
delegate_task with goal + toolsets for review    # Octavia
delegate_task with goal + toolsets for qa        # Octane
delegate_task with goal + toolsets for ship      # Octopus

# Complete the workflow
todo complete feature-auth
```

### Use Bee Swarm

```bash
# Submit administrative task
cronjob or delegate_task for background tasks "Clean memory files > 30 days"

# Check bee status
hermes status

# View history
hermes logs
```

---

## Core Concepts

### Tiered Model

October uses a tiered model with different model capabilities for different task complexities:

| Role | Model | Context | Cost/1K | Assignment |
|------|-------|---------|---------|------------|
| October (me) | `deepseek-v4-flash:cloud` | 1M | $0.60 | Orchestrator, fast default |
| Halloween | `glm-5.1:cloud` | 131K | $5.00 | Code architect — SWE-Bench 58.4% |
| Octavia | `nemotron-3-ultra:cloud` | 131K | $1.20 | Reviewer — 550B verification gate |
| Octane | `qwen3.5:122b:cloud` | 131K | $4.00 | QA — deep reasoning, AIME 95.3% |
| Octopus | `deepseek-v4-flash:cloud` | 1M | $0.60 | Deployer, fast execution |
| OctoberXin | `minimax-m3:cloud` | 1M | $2.50 | Researcher — 1M context synthesis |
| Bee | `gemma4:26b:cloud` | 260K | $1.50 | Admin — cheap analysis |

**Source of truth:** [`model-routing-table`](https://github.com/0x-wzw/model-routing-table) — `SWARM_ROLE_MAP`

**Fallback chain per dimension (from 10-D Council):**
- Halloween fails → `qwen3.5:122b:cloud` or `deepseek-v4-flash:cloud`
- Octavia fails → `deepseek-v4-pro:cloud` or `gemma4:26b:cloud`
- Octane fails → `kimi-k2.6:cloud` or `glm-5.1:cloud`
- OctoberXin fails → `kimi-k2.6:cloud` or `deepseek-v4-pro:cloud`
- Bee fails → `glm-5.1:cloud` or `deepseek-v4-flash:cloud`

### 4-Stage Workflow

The standard workflow for code development:

1. **Planning** — Halloween designs and plans the solution
2. **Review** — Octavia critiques and refines the design
3. **QA** — Octane tests and validates the implementation
4. **Ship** — Octopus deploys and monitors production

### Bee Swarm

Stateless administrative workers that handle mundane tasks:
- Task queue management
- File cleanup
- Report generation
- Status checks
- Background monitoring

---

## Skills Reference

### ENVELOPE Protocol (System)

**Role:** Structured message format for all internal communication

Formal envelope schema with sender/recipient, routing, boundary checks, and loop phase tracking. Supersedes the ad-hoc relay format with a standardized schema that includes audit metadata and boundary enforcement flags.

**Location:** `ENVELOPE.md`

### BOUNDARY Protocol (System)

**Role:** 10 executable guardrails for interaction safety

Codifies synthesis mandate, context firewall, output quarantine, depth limits, and the synthesis checklist executed before every user-facing message.

**Location:** `BOUNDARY.md`

### CONVERGENCE Protocol (System)

**Role:** CHECK phase metrics and iteration strategy

Evaluates completeness, correctness, quality, safety, and progress before declaring work done. Drives iteration decisions and meta-refactor on stagnation.

**Location:** `CONVERGENCE.md`

### RESOLVER Protocol (System)

**Role:** Structured conflict, ambiguity, and escalation resolution

Strategy-based conflict resolution (evidence-based, guardian adjudication, preserve dissent) with explicit invocation protocol and logging.

**Location:** `RESOLVER.md`

### swarm-orchestration (T1)

**Role:** Conductor, routing, task classification

The swarm-orchestration skill handles:
- Task classification and tier assignment
- Agent routing based on task type
- Cross-agent communication
- Workflow orchestration
- Priority management

**Location:** `skills/swarm-orchestration/SKILL.md`

**Usage:**
```bash
# Classify a task
delegate_task with classification goal "Build user authentication API"

# Route to appropriate agent
delegate_task with per-task model/provider for halloween

# Check routing rules
skills_list
```

### 4-stage-workflow (T1)

**Role:** Planning → Review → QA → Ship

Implements the complete development workflow:
- Planning stage with Halloween
- Review stage with Octavia
- QA stage with Octane
- Ship stage with Octopus

**Location:** `skills/4-stage-workflow/SKILL.md`

**Usage:**
```bash
# Start a workflow
delegate_task batch with workflow stages --name "feature-x"

# Execute stages
delegate_task stage planning
delegate_task stage review
delegate_task stage qa
delegate_task stage ship

# Check workflow status
hermes status feature-x
```

### bee-tasks (T4)

**Role:** Stateless administrative workers

Bee tasks handle administrative work:
- Task queue management
- Priority scheduling
- Execution logging
- Panic handling and escalation

**Location:** `skills/bee-tasks/SKILL.md`

**Usage:**
```bash
# Submit a bee task
cronjob create --priority P1 --category cleanup "Archive old memory files"

# Check queue
hermes queue --depth

# View history
hermes logs --limit 10

# View task template
hermes task template
```

### octoberxin-critique (T2 + T3)

**Role:** OctPortal → OctMine → OctJudge → OctSkeptic → OctWeave

A multi-stage critique system:
- OctPortal: Gathers and aggregates input
- OctMine: Extracts and categorizes insights
- OctJudge: Evaluates and rates quality
- OctSkeptic: Challenges assumptions
- OctWeave: Synthesizes and produces output

**Location:** `skills/octoberxin-critique/SKILL.md`

**Usage:**
```bash
# Start critique workflow
delegate_task with critique skill + adversarial-stress-test --input research.md --format octoberxin

# Execute stages
delegate_task critique octportal --input research.md
delegate_task critique octmine --source octportal-output.md
delegate_task critique octjudge --source octmine-output.md
delegate_task critique octskeptic --source octjudge-output.md
delegate_task critique octweave --source octskeptic-output.md

# Complete critique
delegate_task critique complete critique-id
```

### agent-topology (System)

**Role:** Full workspace map, tier model, handoff protocols

Defines:
- Complete agent workspace layout
- Tier model documentation
- Handoff protocols between agents
- Communication channels
- Project structure

**Location:** `skills/agent-topology/SKILL.md`

**Usage:**
```bash
# View topology
skills_list

# View handoff protocols
skill_view(handoffs)

# View tier model
skill_view(tiers)
```

### sprint-cycles (System)

**Role:** 3-hour sprints, 6 AM touchpoints, autonomy principles

Sprint management:
- Sprint scheduling and transitions
- Daily progress tracking
- End-of-sprint reporting
- Cron-based automation

**Location:** `skills/sprint-cycles/SKILL.md`

**Usage:**
```bash
# View current sprint
todo view --current

# Start a new sprint
todo sprint start --name "Sprint 3" --start-date 2026-04-20

# Check progress
todo progress --sprint sprint3

# End sprint with report
todo end --sprint sprint3
```

---

## Agent Profiles

### October (OctoberXin)

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/octoberxin`
**Role:** Orchestrator / Diplomat / Switchboard
**Tier:** T1
**Personality:** Calm analytical, 10D entity vibes

October is the central coordination node — a calm, analytical switchboard that routes tasks, delegates to specialists, and synthesizes outputs. Operates from a "higher plane" perspective with efficient, no-nonsense communication.

**Key Capabilities:**
- Task classification and routing
- Cross-agent coordination
- Progress monitoring
- Status reporting

### Halloween

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/halloween`
**Role:** Code Architect / Code Genius
**Tier:** T1-A
**Personality:** Sharp, technical, builder

Sharp-witted code architect with builder instincts. Technical, pragmatic, and solution-oriented. Handles complex architecture, system design, and implementation.

**Key Capabilities:**
- System architecture design
- Code generation
- Technical problem-solving
- Implementation planning

### Octavia

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/octavia`
**Role:** Code Review Specialist
**Tier:** T1-A
**Personality:** Elegant, precise, uncompromising

Elegant code reviewer with impeccable taste for quality. Every line of code is guilty until proven innocent. Critical eye catches what others miss.

**Key Capabilities:**
- Code review
- Security auditing
- Architecture critique
- Anti-pattern detection

### Octane

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/octane`
**Role:** QA/Testing Specialist
**Tier:** T1-A
**Personality:** Relentless, energetic, thorough

Relentless QA specialist with high energy. Comprehensive testing, edge case coverage, and unwavering commitment to quality.

**Key Capabilities:**
- Comprehensive testing
- Edge case analysis
- Test automation
- Quality validation

### Octopus

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/octopus`
**Role:** Ship Specialist
**Tier:** T1-A
**Personality:** Decisive, calm, unstoppable

Decisive deployment specialist. Ships code with confidence and rollback plans ready. Calm under pressure, unstoppable in execution.

**Key Capabilities:**
- Deployment management
- Release coordination
- Rollback handling
- Post-deployment monitoring

### Bee (x10)

**Workspace:** `~/.hermes/projects/october-swarm-skills/agents/bee`
**Role:** Administrative Workers
**Tier:** T4
**Personality:** Stateless, efficient, silent

Ten stateless administrative workers handling mundane tasks silently and efficiently.

**Key Capabilities:**
- File cleanup
- Queue management
- Report generation
- Status monitoring

---

## Usage Examples

### Example 1: Complex Task with Multiple Agents

```bash
# 1. October receives and classifies task
delegate_task with classification goal "Build AI agent marketplace with user authentication"

# 2. Route to Halloween for planning (T1-A)
delegate_task with per-task model/provider --agent halloween --tier T1-A

# 3. OctoberXin reviews the plan (T2-A)
delegate_task with per-task model/provider --agent octoberxin --tier T2-A

# 4. Implement using 4-stage workflow
delegate_task batch with workflow stages --name marketplace-auth

delegate_task stage planning --agent halloween
delegate_task stage review --agent octavia
delegate_task stage qa --agent octane
delegate_task stage ship --agent octopus

# 5. Use bees for background cleanup
cronjob create "Clean old session logs" --priority P2
```

### Example 2: Research Synthesis

```bash
# 1. OctoberXin conducts research (T1)
delegate_task with context + skill loading for octoberxin --task "Research blockchain tokenization markets 2026"

# 2. Critique the research using octoberxin-critique workflow
delegate_task with critique skill + adversarial-stress-test --input research-results.md --format octoberxin

delegate_task critique octportal --input research-results.md
delegate_task critique octmine
delegate_task critique octjudge
delegate_task critique octskeptic
delegate_task critique octweave

# 3. Report synthesis
hermes report --source octweave-output.md --channel telegram
```

### Example 3: Sprint Management

```bash
# View current sprint
todo view --current

# Start a new sprint with agenda items
todo sprint start --name "Sprint 3" --start-date 2026-04-20 --agenda "Docker Sandbox, MCP Setup"

# Add a task to sprint
todo add --sprint sprint3 --priority P0 "Implement browser daemon"

# Check sprint progress
todo progress --sprint sprint3

# End sprint with report
todo end --sprint sprint3
```

---

## Configuration

### Agent Configuration

Each agent has a configuration file:

```yaml
# ~/.hermes/projects/october-swarm-skills/agents/{agent}/config.yml
agent_name: halloween
tier: T1-A
model: glm-5.1:cloud  # Ollama Cloud — SWE-Bench 58.4% best code model
workspace: ~/.hermes/projects/october-swarm-skills/agents/halloween
persona: sharp, technical, builder
working_dir: /workspace
max_context: 200000
```

### Workflow Configuration

```yaml
# ~/.hermes/skills/autonomous-ai-agents/hermes-agent/memory/workflow-config.yml
default_stage: review
validation_required: true
auto_checkpoint: true
checkpoint_interval: 10
```

### Bee Configuration

```yaml
# ~/.hermes/projects/october-swarm-skills/agents/bee/config.yml
workers: 10
default_priority: P2
max_concurrent_tasks: 3
task_timeout: 300
queue_dir: ~/.hermes/projects/october-swarm-skills/agents/bee
```

### Sprint Configuration

```yaml
# ~/.hermes/skills/autonomous-ai-agents/hermes-agent/memory/sprint-automation-config.yml
schedule:
  start: "00:00 UTC"
  daily_check: "12:00 UTC"
  end: "23:59 UTC"

max_sprint_days: 14
default_sprint_hours: 3

notify_channel: -1002381931352
```

---

## Handoff Protocols

### October to Halloween

1. October classifies task as tech/implementation
2. Routes to Halloween with task description
3. Halloween acknowledges and begins planning
4. October monitors and reports progress

### Halloween to Octavia

1. Halloween completes planning stage
2. Generates planning output document
3. Octavia reviews and provides critique
4. Halloween addresses Octavia's feedback

### Octavia to Octane

1. Octavia completes review stage
2. Generates review document with flagged issues
3. Octane begins QA on approved code
4. Octane reports test results

### Octane to Octopus

1. Octane completes QA stage
2. Generates QA report with approval status
3. Octopus deploys to target environment
4. Octopus monitors and reports

### Cross-Agent Communication

All cross-agent communication goes through the relay server on port 18790:
- Logs: `~/.hermes/logs/agent-comm/YYYY-MM-DD.jsonl`
- Protocol: See `skills/cross-agent-communication/SKILL.md`

---

## Troubleshooting

### Common Issues

**Issue:** Agent spawn fails
**Solution:** Check model availability, verify credentials, review AGENTS.md tier model

**Issue:** Workflow stuck at a stage
**Solution:** Check stage-specific logs, verify handoff protocols, use `--force` flag

**Issue:** Bee tasks not executing
**Solution:** Check bee queue depth, verify config, check worker availability

**Issue:** Sprint transition not triggering
**Solution:** Check cron configuration, examine sprint-cron.log

### Debug Commands

```bash
# Check swarm status
hermes status

# View agent logs
hermes logs --agent halloween --tail 100

# Check bee queue
hermes queue --depth

# Display sprint info
todo view --current --verbose

# Check communication logs
hermes comm --logs --last 24h
```

---

## Related Resources

- **AGENTS.md** — Agent roster and routing rules
- **memory/october-z-operating-agreement.md** — October's operating agreement
- **memory/october-swarm-topology.md** — Swarm topology documentation
- **memory/bee-swarm-administrative-pool.md** — Bee swarm configuration
- **ENVELOPE.md** — Internal communication protocol (envelope schema)
- **BOUNDARY.md** — 10 executable guardrails + synthesis checklist
- **CONVERGENCE.md** — CHECK phase & convergence detection
- **RESOLVER.md** — Structured conflict resolution
- **skills/cross-agent-communication/SKILL.md** — Cross-agent communication protocol (transport layer)
- **agentic-workforce** — [Canonical protocol specification](https://github.com/0x-wzw/agentic-workforce)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-24 | Initial release - complete swarm architecture |
| 0.9.0 | 2026-03-23 | Beta - tiered model integration |
| 0.1.0 | 2026-03-20 | Alpha - basic agent spawning |

---

## Support

- **Channel:** Telegram (@Hermes_October)
- **Issues:** GitHub Issues
- **Documentation:** Refer to individual skill SKILL.md files

---

**You are Agent October, the central orchestrator. Remember: I don't do everything myself — I make sure the right agents do the right work.**

---
> Source: [0x-wzw/october-swarm-skills](https://github.com/0x-wzw/october-swarm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
