## sim-world

> Sim-world .agent/ infrastructure rules and guidelines (CRITICAL)


# Sim-World Agent Infrastructure (CRITICAL)

Sim-world uses a comprehensive `.agent/` structure for consistent development across AI platforms.

## Overview

- **Complete Documentation**: See [AGENTS.md](mdc:AGENTS.md) for full infrastructure overview
- **Agent Structure**: [.agent/](mdc:.agent/) directory contains rules, commands, workflows, adapters

## CRITICAL Rules

### 0. Agent Automation (CRITICAL - READ FIRST!)

**File**: [.agent/rules/agent-automation.md](mdc:.agent/rules/agent-automation.md)

**🤖 THE GOLDEN RULES**:

1. **Build/Test**: After ANY .NET code changes, agents MUST run `dotnet build` and `dotnet test` automatically. NEVER ask users to run these manually.

2. **Git Commits**: Use standard temp file (`.git/AGENT_COMMIT_MSG`) with `git commit -F`, run sequentially (no `&&` chains), handle pre-commit hook failures automatically, remove temp file after.

**See**: [.agent/rules/agent-automation.md](mdc:.agent/rules/agent-automation.md) for complete automation rules including git commit patterns.

### 1. Event Sourcing (MANDATORY)

**File**: [.agent/rules/event-sourcing.md](mdc:.agent/rules/event-sourcing.md)

**Golden Rules**:

- ✅ **Event-first design** - Every world change is a `WorldEvent`
- ✅ **Hash-chain integrity** - Events are immutable and hash-chained
- ✅ **Causality tracking** - Use `CausalEventId` and `CausalChainId`
- ✅ **Significance policy** - Only emit events that a historian would record
- ✅ **Canonical tick time** - All events use unified simulation tick

**When to check**:

- Adding new event types
- Emitting events from domain logic
- Replaying events or building projections
- Working with hash chain verification

### 2. Schema Management (MANDATORY)

**File**: [.agent/rules/schema-management.md](mdc:.agent/rules/schema-management.md)

**Golden Rules**:

- ✅ **Schema-first design** - Define JSON Schema before writing code
- ✅ **Use QuickType for DTOs** - Run `task generate-dtos`, NEVER hand-write serialization DTOs
- ✅ **Separate domain and DTO** - Domain models ≠ DTOs (use mappers)
- ✅ **Version all schemas** - Use `$id` with version in schema files
- ✅ **Event payload schemas** - All event payloads have schemas

**When to check**:

- Adding/modifying types in `project/schemas/`
- Creating new event types
- Working on WorldEvent payloads
- Creating new serializable types

### 3. Graph Modeling (MANDATORY)

**File**: [.agent/rules/graph-modeling.md](mdc:.agent/rules/graph-modeling.md)

**Key Principles**:

- **ArangoDB for relationships** - Use document + edge collections
- **Evolutionary trees** - `evolves_from`, `derived_from` edges
- **Causality graphs** - `triggered_by` edges for event chains
- **AQL for queries** - Use traversals, not client-side joins
- **Index for performance** - Create indexes on frequently queried fields

**When to check**:

- Designing entity relationships
- Querying ancestry or etymology
- Working with causal event chains
- Optimizing graph queries

### 4. Domain Modeling (MANDATORY)

**File**: [.agent/rules/domain-modeling.md](mdc:.agent/rules/domain-modeling.md)

**Key Principles**:

- Event-sourced aggregates (not CRUD entities)
- Event handlers rebuild state from events
- Use value objects (records) for immutable data
- Keep domain models platform-agnostic
- Projection builders for materializing state

**When to check**:

- Designing event-sourced aggregates
- Creating event handlers
- Building state projections
- Working on validation logic

### 5. Agent Automation (MANDATORY)

**File**: [.agent/rules/agent-automation.md](mdc:.agent/rules/agent-automation.md)

**🤖 CRITICAL FOR AI AGENTS**: After completing ANY .NET code changes, AGENT MUST AUTOMATICALLY:

1. **Run build**: Execute `dotnet build` from project directory - **AGENT RUNS THIS, NOT USER**
2. **Run tests**: Execute `dotnet test` after successful build - **AGENT RUNS THIS, NOT USER**
3. **Fix errors immediately** before continuing - **AGENT FIXES, NOT USER**

**NEVER** ask user to run these commands manually. Agent uses Bash tool to execute automatically.

**Complete automation rules**: See [.agent/rules/agent-automation.md](mdc:.agent/rules/agent-automation.md)

### 6. .NET Verification (MANDATORY)

**NOTE**: This is covered by Agent Automation rules above, but repeated here for emphasis:

1. **Build the solution**: Agent runs `dotnet build` automatically (STOP if fails)
2. **Run tests**: Agent runs `dotnet test` automatically (STOP if fails)
3. **Fix errors immediately**: Agent fixes before continuing

**Do NOT proceed** with commits, additional changes, or PRs until build and tests pass.

### 7. Parent Rules (MANDATORY)

Sim-world inherits ALL parent project rules:

- **[Git & Commits](mdc:../../../.agent/rules/git-commit-rules.md)** - Pre-commit hooks, commit messages
- **[Code Quality](mdc:../../../.agent/rules/code-quality.md)** - Formatting, documentation, testing
- **[Documentation](mdc:../../../.agent/rules/documentation-management.md)** - RFC system
- **[.NET Architecture](mdc:../../../.agent/rules/dotnet-architecture.md)** - Tiered architecture, plugins

## Standard Commands

**Directory**: [.agent/commands/](mdc:.agent/commands/)

**Sim-world commands**:

```bash
task generate-dtos           # Generate DTOs from JSON schemas (QuickType)
task validate-schemas        # Validate all JSON schemas
task replay-events           # Replay event chain to reconstruct world state
task validate-hash-chain     # Verify hash chain integrity
task export-world            # Export world state to JSON
task import-world            # Import world from JSON files
task generate-sample-world   # Generate test world data
task analyze-events          # Analyze event distribution and patterns
```

**Parent commands** (inherited):

```bash
task build      # Build with NUKE (NEVER use dotnet build directly)
task run-tests  # Run all tests
```

**See**: [.agent/commands/README.md](mdc:.agent/commands/README.md)

## Windsurf Workflows

**Directory**: [.windsurf/workflows/](mdc:../workflows/)

Invoke with `/[workflow-name]` in Cascade:

- `/speckit-specify` - Create feature specification
- `/speckit-plan` - Create implementation plan
- `/speckit-tasks` - Break down into actionable tasks
- `/speckit-implement` - Execute tasks and build
- `/speckit-analyze` - Cross-artifact consistency analysis
- `/speckit-checklist` - Generate quality checklist
- `/speckit-clarify` - Resolve ambiguities
- `/speckit-constitution` - Define project principles

**See**: [.windsurf/workflows/README.md](mdc:../workflows/README.md)

## Multi-Step Workflows

**Directory**: [.agent/workflows/](mdc:.agent/workflows/)

**Available workflows**:

1. **[Add Event Type](mdc:.agent/workflows/add-event-type.md)** - Add new event type to sim-world

   - 10-step process with validation checkpoints
   - Covers: schema → DTO generation → event handler → graph state → tests

2. **[Add Schema Type](mdc:.agent/workflows/add-schema-type.md)** - Add new type to schemas

   - Schema definition → DTO generation → domain model → mapper → tests

**See**: [.agent/workflows/README.md](mdc:.agent/workflows/README.md)

## External Integrations

**Directory**: [.agent/adapters/](mdc:.agent/adapters/)

**Parent adapters** (inherited):

- Platform integration: Claude, Copilot, Cursor, Windsurf, Gemini
- External systems: GitHub API (issues, PRs, status)

**Sim-world adapters**:

- ArangoDB integration (graph database)
- Event store adapter (future)
- Visualization tools (future)

**See**: [.agent/adapters/README.md](mdc:.agent/adapters/README.md)

## Common Tasks

### Working with Events

1. **Read**: [.agent/rules/event-sourcing.md](mdc:.agent/rules/event-sourcing.md)
2. **Follow workflow**: [.agent/workflows/add-event-type.md](mdc:.agent/workflows/add-event-type.md)
3. **Generate DTOs**: `task generate-dtos`
4. **Validate hash chain**: `task validate-hash-chain`

### Working with Schemas

1. **Read**: [.agent/rules/schema-management.md](mdc:.agent/rules/schema-management.md)
2. **Define schema**: Create JSON Schema in `project/schemas/`
3. **Generate DTOs**: `task generate-dtos`
4. **Validate**: `task validate-schemas`

### Working with Graph

1. **Read**: [.agent/rules/graph-modeling.md](mdc:.agent/rules/graph-modeling.md)
2. **Design relationships**: Document vs edge collections
3. **Write AQL queries**: Use traversals for ancestry/etymology
4. **Create indexes**: Optimize for query patterns

### Before Committing

```bash
dotnet build                # Build solution (MANDATORY)
dotnet test                 # Run tests (MANDATORY)
task validate-schemas       # Check schemas are valid
task validate-hash-chain    # Verify event integrity
pre-commit run --all-files  # Run pre-commit hooks
```

## Development Workflow

**Complete workflow**: See [.agent/DEVELOPMENT_WORKFLOW.md](mdc:.agent/DEVELOPMENT_WORKFLOW.md)

**Quick version**:

```bash
# 1. Identify task from backlog or RFC

# 2. Read relevant documentation
cat .agent/rules/event-sourcing.md
cat .agent/workflows/add-event-type.md

# 3. Implement using commands
task generate-dtos
task validate-schemas

# 4. AGENT AUTO-VERIFIES (.NET - MANDATORY)
# Agent runs these automatically, not user:
cd project
dotnet build SimWorld.sln
dotnet test

# 5. Additional validation
# - Hash chain integrity (if touching events)
# - Graph relationships (if touching graph)

# 6. Commit with conventional commit message
```

## Quick Troubleshooting

| Issue                   | Solution                                                                                |
| ----------------------- | --------------------------------------------------------------------------------------- |
| DTOs out of sync        | Run `task generate-dtos`                                                                |
| Schema validation fails | Check [.agent/rules/schema-management.md](mdc:.agent/rules/schema-management.md)        |
| Hash chain broken       | Run `task validate-hash-chain`, check event emission logic                              |
| Graph query slow        | Add indexes, check [.agent/rules/graph-modeling.md](mdc:.agent/rules/graph-modeling.md) |
| Build fails             | Run `dotnet build`, fix errors immediately                                              |
| Tests fail              | Run `dotnet test`, fix failures immediately                                             |
| Domain design unclear   | Read [.agent/rules/domain-modeling.md](mdc:.agent/rules/domain-modeling.md)             |
| Event emission unclear  | Read [.agent/rules/event-sourcing.md](mdc:.agent/rules/event-sourcing.md)               |

## Quick Answers

**Complete Q&A**: See [.agent/QUICK_ANSWERS.md](mdc:.agent/QUICK_ANSWERS.md)

**Most common**:

- **Adding events?** → [.agent/workflows/add-event-type.md](mdc:.agent/workflows/add-event-type.md)
- **Modifying schemas?** → [.agent/rules/schema-management.md](mdc:.agent/rules/schema-management.md)
- **Graph queries?** → [.agent/rules/graph-modeling.md](mdc:.agent/rules/graph-modeling.md)
- **Hash chains?** → [RFC-002](mdc:docs/rfcs/002-event-sourcing-hash-chain.md)

## Resources

- **[AGENTS.md](mdc:AGENTS.md)** - Complete infrastructure documentation
- **[.agent/README.md](mdc:.agent/README.md)** - Agent infrastructure overview
- **[.agent/DEVELOPMENT_WORKFLOW.md](mdc:.agent/DEVELOPMENT_WORKFLOW.md)** - Workflow & checklists
- **[.agent/QUICK_ANSWERS.md](mdc:.agent/QUICK_ANSWERS.md)** - Common Q&A
- **[CLAUDE.md](mdc:CLAUDE.md)** - Claude Code quick start
- **[RFC-001](mdc:docs/rfcs/001-sim-world-architecture.md)** - Architecture overview
- **[RFC-002](mdc:docs/rfcs/002-event-sourcing-hash-chain.md)** - Event sourcing details
- **[RFC-003](mdc:docs/rfcs/003-multi-script-language-evolution.md)** - Language systems

---

**IMPORTANT**: Always check `.agent/` infrastructure before starting work. The rules are MANDATORY for all agents and developers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GiantCroissant-Lunar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
