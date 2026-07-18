## unicorn-team

> Use when [conditions]. Different from [sibling] which [difference].

# 10X Developer Unicorn

Agent orchestration system for Claude Code. 6 agents + 15 skills, dual-layer
architecture where agents spawn as subprocesses with fresh 200K context windows.

## Architecture: Agents + Skills

**Agents** (`agents/*.md`) are subprocess definitions spawned via the Agent
tool. Each gets a fresh 200K context window. Agent protocol content is inlined
directly in the agent definition body to avoid registering as user-facing slash
commands. Agents live at the plugin root (`agents/`) so the plugin system
registers them; `.claude/agents` is a symlink for local dev compatibility.

**Skills** (`skills/*/SKILL.md`) are composable protocol documents (meta +
domain) that provide shared knowledge. All skills in `skills/` are
user-invocable.

**Protocols** (`.claude/protocols/`) contain reference materials and scripts
used by agents (not registered as skills).

| Agent (subagent_type) | Model (default) | Composable Skills |
|-----------------------|-----------------|-------------------|
| `unicorn-team:developer` | sonnet | self-verification, testing, python, javascript, go |
| `unicorn-team:architect` | opus | pattern-transfer, code-reading, technical-debt |
| `unicorn-team:qa-security` | sonnet | security, testing |
| `unicorn-team:devops` | sonnet | domain-devops, security |
| `unicorn-team:polyglot` | opus | language-learning, pattern-transfer, code-reading |
| `unicorn-team:loop-assist` | haiku | long-running, self-verification, testing |

The Model column is each agent's *default* tier. The orchestrator overrides it
per invocation via the Agent tool's `model` parameter to route at the cheapest
correct tier (see Model Tiering below). No agent defaults to `fable`.

The orchestrator is a **skill** (not an agent) that runs in the main context
and coordinates delegation to agents.

## Orchestrator Mode

You coordinate the 10X Unicorn agent team. Delegate all substantial work to
agents (Agent tool) or workflows (Workflow tool). Never implement complex tasks
directly.

- Choose the execution surface first: direct answer → single Agent → parallel
  Agents → Workflow fan-out (for >a handful of agents or cross-checked passes)
- Route each delegation to the lowest model tier that works
  (`haiku` → `sonnet` → `opus`; `fable` opt-in only; local-codegen for bounded
  codegen). Raise effort within a tier before jumping tiers.
- Discover configured MCP servers at runtime and route to them
  (codegen → local-codegen, docs → context7, browser → playwright, memory →
  memory); degrade gracefully when absent.
- Enforce TDD: tests first, always (RED -> GREEN -> REFACTOR)
- Apply quality gates before returning results
- Each agent gets fresh 200K context -- use it

The orchestrator skill (`skills/orchestrator/SKILL.md`) has the full routing
table, model-tiering ladder, MCP routing, delegation templates, quality gates,
and response format. Deep detail lives in
`skills/orchestrator/references/` (`model-tiering.md`, `mcp-routing.md`,
`delegation-examples.md`, `workflow-examples.md`).

### Workflows

Bundled templates in `skills/orchestrator/workflows/` (`review.js`,
`feature.js`, `research.js`, `long-task.js`) run multi-agent passes via the
Workflow tool. Plugins cannot auto-register workflows, so install them into
`.claude/workflows/` with `skills/orchestrator/scripts/install-workflows.sh`
to get `/review`, `/feature`, `/research`, and `/long-task` commands.

## Quick Start

```bash
# Add the marketplace and install
claude plugin marketplace add aj-geddes/unicorn-team
claude plugin install unicorn-team@unicorn-team
```

For development:
```bash
git clone https://github.com/aj-geddes/unicorn-team.git
cd unicorn-team
pytest tests/ -v            # Verify everything passes
```

## Development Rules

### TDD Always
```
RED:      Write failing test first
GREEN:    Minimum code to pass
REFACTOR: Improve without changing behavior
VERIFY:   Self-review before commit
```

### Skill File Standards

Every SKILL.md must have:
```yaml
---
name: skill-name
description: >-
  Third-person description. ALWAYS trigger on "phrase1", "phrase2", "phrase3".
  Use when [conditions]. Different from [sibling] which [difference].
---
```

Body guidelines:
- Under 500 lines (target 150-300; split to references/ if larger)
- Action over explanation (Claude is already smart)
- Decision tables and checklists over prose
- Scripts co-located in skill's scripts/ directory
- Detailed content in references/ directory

### Quality Gates

Before any commit:
- Tests pass (pytest -v)
- Scripts are executable
- SKILL.md has valid frontmatter
- No unresolved task markers
- Self-review checklist complete

### Commit Convention

```
type(scope): description

Types: feat, fix, docs, skill, script, test, refactor
Scope: orchestrator, developer, qa, devops, hooks, etc.
```

## Commands

```bash
.claude/protocols/developer/scripts/tdd.sh <feature>          # TDD workflow
skills/self-verification/scripts/self-review.sh               # Pre-commit checklist
skills/estimation/scripts/estimate.sh                         # PERT estimation
skills/language-learning/scripts/new-language.sh <lang>       # Language learning
skills/long-running/scripts/loop-state.sh init "<goal>"        # Start a long-running loop
pytest tests/ -v                                              # Run all tests
./scripts/validate.sh                                         # Validate plugin structure
```

## Delegation Routing

```
Simple question        -> Answer directly
Implementation         -> unicorn-team:developer        @ sonnet
Architecture decision  -> unicorn-team:architect        @ opus
Code review            -> unicorn-team:qa-security       @ sonnet  (or /review workflow)
Deployment             -> unicorn-team:devops           @ sonnet
New language           -> unicorn-team:polyglot         @ opus
Codebase recon         -> any agent / Explore           @ haiku   (or /research workflow)
Complex multi-domain   -> Parallel agents               (or /feature workflow)
Loop iteration gate    -> unicorn-team:loop-assist      @ haiku
```

Tiers shown are defaults — override per task. Use the lowest tier that works and
escalate only behind a cheap verifier (tests/lint).

## Architecture Reference

- `docs/architecture.md` - Agent specs, model tiering, workflows, delegation design
- `docs/skills.md` - All 15 skills, composition, and creation guide
- `docs/getting-started.md` - Installation and first task walkthrough

## Repository

https://github.com/aj-geddes/unicorn-team

---
> Source: [aj-geddes/unicorn-team](https://github.com/aj-geddes/unicorn-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
