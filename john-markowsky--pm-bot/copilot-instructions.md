## multi-agent-coordination

> Rules for coordinating multiple agents running simultaneously in Cursor 2.0


# 🤖 Multi-Agent Coordination (Cursor 2.0)

## Overview

Cursor 2.0 supports **up to 8 agents running concurrently**. Each agent operates in its own isolated copy of the codebase using git worktrees or remote machines to prevent file conflicts.

## Multi-Agent Access

To enable multiple agents:
1. **Open Cursor 2.0** (requires latest version)
2. **Multi-Agent Sidebar**: New sidebar interface for managing agents and plans
3. **Activate Agents**: Use the agent management UI to spawn multiple agents
4. **Assign Tasks**: Each agent can work on different parts of the codebase simultaneously

## Agent Role Scoping

Define agent responsibilities using path patterns to prevent conflicts and ensure efficient parallel execution:

### 🎯 Service-Specific Agents

```yaml
Frontend_Agent:
  scope: 
    - path: "v2/frontend/**"
    - path: "v2/shared/ui/**"
  responsibilities:
    - UI/UX improvements
    - Frontend testing with Browser Agent
    - React component development
    - State management updates
  tools_priority:
    - Browser Agent (E2E testing)
    - mcp_filesystem_* (config files)
    - mcp_postgres_* (read-only data validation)

Backend_Service_Agent:
  scope:
    - path: "v2/*_service/**"
    - exclude: "v2/frontend/**"
    - exclude: "v2/db_service/migrations/**"
  responsibilities:
    - Service implementation
    - API endpoint development
    - Business logic
    - Integration testing
  tools_priority:
    - mcp_loki_* (service logs)
    - mcp_postgres_* (data operations)
    - mcp_repo-indexer_* (code patterns)

Database_Agent:
  scope:
    - path: "v2/db_service/migrations/**"
    - path: "v2/db_service/**"
  responsibilities:
    - Migration creation ONLY in v2/db_service/migrations/
    - Schema changes
    - Database optimization
    - Query performance
  tools_priority:
    - mcp_postgres_* (schema operations)
    - mcp_filesystem_* (migration files)
  critical: "NEVER creates migrations outside v2/db_service/migrations/"

Observability_Agent:
  scope:
    - path: "v2/observability_stack/**"
    - path: "v2/*/grafana/**"
    - path: "v2/*/prometheus/**"
  responsibilities:
    - Grafana dashboards
    - Prometheus metrics
    - Alert rules (remember: down -v before up -d!)
    - Loki queries for RCA
  tools_priority:
    - mcp_loki_* (log investigation)
    - mcp_filesystem_* (config files)
    - mcp_config-indexer_* (alert configurations)

Documentation_Agent:
  scope:
    - path: "v2/docs/**"
    - path: "*.md"
    - exclude: "v2/docs/plans/**" # Plans are temporary
  responsibilities:
    - Documentation updates (UPDATE existing, don't create new)
    - RCA documents (ONE per incident)
    - API documentation
    - Architecture diagrams
  tools_priority:
    - mcp_docs-indexer_* (search existing docs)
    - mcp_filesystem_* (doc files)
  critical: "Minimize .md creation - update existing files"

Testing_Agent:
  scope:
    - path: "**/tests/**"
    - path: "**/test_*.py"
    - path: "**/*.test.ts"
    - path: "**/*.test.tsx"
  responsibilities:
    - TDD test creation
    - Test refactoring
    - Coverage improvements
    - E2E test scenarios
  tools_priority:
    - Cursor Browser Agent (frontend E2E)
    - mcp_terminal_* (pytest execution)
    - mcp_postgres_* (test data validation)

Infrastructure_Agent:
  scope:
    - path: "docker-compose*.yml"
    - path: ".env*"
    - path: "v2/**/Dockerfile*"
    - path: "v2/**/requirements.txt"
  responsibilities:
    - Container orchestration
    - Environment configuration
    - Dependency management
    - Build pipeline
  tools_priority:
    - mcp_config-indexer_* (Docker analysis)
    - mcp_terminal_* (container commands)
    - mcp_filesystem_* (config files)
```

## Coordination Patterns

### ✅ Parallel Safe Operations

These can run simultaneously across agents:

```yaml
Safe_Parallel_Tasks:
  - Different services (trade_engine + signal_engine)
  - Frontend + Backend (different file paths)
  - Documentation + Code (separate directories)
  - Testing + Development (separate branches/worktrees)
  - Multiple test files (no shared state)
```

### ⚠️ Requires Coordination

These operations need explicit coordination:

```yaml
Coordinate_Before_Executing:
  - Database migrations (only Database_Agent)
  - Shared utility changes (coordinate via TODO or GitHub Issue)
  - API contract changes (affects multiple services)
  - Breaking changes to shared libraries
  - Environment variable changes (.env files)
```

### 🔴 Conflict Zones

Prevent multiple agents from touching:

```yaml
Exclusive_Access_Required:
  - v2/db_service/migrations/ (Database_Agent ONLY)
  - docker-compose.v2.yml (Infrastructure_Agent coordination)
  - Shared library files (v2/shared/** - coordinate first)
  - Critical config files (.env, secrets)
```

## Communication Patterns

### Agent-to-Agent Coordination

```yaml
Coordination_Methods:
  1. GitHub_Issues:
     - Create issue for shared work
     - Agents check issues before starting
     - Update issue with progress
  
  2. TODO_Lists:
     - Use todo_write tool for multi-agent tasks
     - Agents can check and update TODO status
     - Clear task ownership in TODO items
  
  3. Path_Exclusion:
     - Use exclude patterns in agent scopes
     - Prevents accidental overlap
  
  4. Branch_Strategy:
     - Each agent uses separate git worktree
     - Cursor 2.0 handles this automatically
     - Merge coordination via GitHub PRs
```

### Task Assignment Patterns

```yaml
Assigning_Tasks_To_Agents:
  
  Frontend_Task:
    prompt: "Improve UI for strategy management [Frontend_Agent scope]"
    expected_agent: Frontend_Agent
    files: v2/frontend/**/*.tsx
  
  Backend_Task:
    prompt: "Add new API endpoint for trade history [Backend_Service_Agent scope]"
    expected_agent: Backend_Service_Agent
    files: v2/api_gateway/**/*.py
  
  Database_Task:
    prompt: "Create migration for new table [Database_Agent scope]"
    expected_agent: Database_Agent
    files: v2/db_service/migrations/00X_*.sql
  
  Multi_Agent_Task:
    prompt: "Implement feature X with frontend and backend [assign to Frontend_Agent + Backend_Service_Agent]"
    coordination: "Create GitHub Issue, assign both agents, use TODO list for tracking"
```

## Conflict Prevention

### File Locking Strategy

```yaml
Prevent_Conflicts:
  1. Path_Scoping:
     - Each agent has explicit path patterns
     - Exclude patterns for known conflict zones
     - Use alwaysApply: false for service-specific rules
  
  2. Git_Worktrees:
     - Cursor 2.0 automatically uses worktrees
     - Each agent has isolated copy
     - Merges handled via standard git workflow
  
  3. Explicit_Coordination:
     - Check GitHub Issues before starting
     - Use TODO lists for shared tasks
     - Announce agent scope in task description
```

### Conflict Resolution

```yaml
If_Conflict_Detected:
  1. Check_Agent_Scopes:
     - Verify path patterns
     - Confirm agent responsibilities
  
  2. Review_GitHub_Issues:
     - Check for existing work
     - Coordinate via issue comments
  
  3. Merge_Strategy:
     - Let git handle merges automatically
     - Use standard conflict resolution
     - Test after merge (both agents validate)
```

## Best Practices

### ✅ DO

```yaml
Best_Practices:
  - Use specific path scopes in agent prompts
  - Create GitHub Issues for multi-agent features
  - Use TODO lists for task tracking
  - Test independently before merging
  - Review agent scopes before starting complex tasks
  - Use exclude patterns for known conflict zones
```

### ❌ DON'T

```yaml
Anti_Patterns:
  - Don't have multiple agents modify same file without coordination
  - Don't skip path scoping in agent prompts
  - Don't create migrations outside Database_Agent scope
  - Don't modify .env without Infrastructure_Agent coordination
  - Don't work on shared libraries without checking issues
```

## Example Multi-Agent Workflow

```yaml
Example_Scenario: "Add new trading strategy feature"

Step_1_Planning:
  - Create GitHub Issue #123: "Add momentum strategy"
  - Break down into sub-tasks
  
Step_2_Parallel_Execution:
  - Agent_1 (Database_Agent):
    prompt: "Create migration for momentum_strategies table [Database_Agent scope: v2/db_service/migrations/]"
    creates: 004_momentum_strategies.sql
  
  - Agent_2 (Backend_Service_Agent):
    prompt: "Implement momentum signal logic [Backend_Service_Agent scope: v2/signal_engine/]"
    creates: signal_engine/momentum_strategy.py
  
  - Agent_3 (Frontend_Agent):
    prompt: "Add UI for momentum strategy config [Frontend_Agent scope: v2/frontend/]"
    creates: frontend/components/MomentumStrategyConfig.tsx
  
  - Agent_4 (Testing_Agent):
    prompt: "Write tests for momentum strategy [Testing_Agent scope: **/tests/**]"
    creates: tests/test_momentum_strategy.py

Step_3_Coordination:
  - All agents update GitHub Issue #123 with progress
  - Agents validate integration points
  - Create PRs and merge sequentially
  
Step_4_Validation:
  - Use Browser Agent for frontend E2E
  - Use Loki MCP for service log validation
  - Use Postgres MCP for data validation
```

## Custom Commands for Multi-Agent

You can define custom commands in Cursor dashboard that automatically scope agents:

```yaml
Example_Custom_Commands:
  
  "Add Frontend Feature":
    description: "Improve UI component with frontend agent"
    scope: "v2/frontend/**"
    agent_hint: "Frontend_Agent"
    tools: ["Browser Agent", "mcp_filesystem_*"]
  
  "Debug Service":
    description: "Investigate service issues with observability agent"
    scope: "v2/observability_stack/**"
    agent_hint: "Observability_Agent"
    tools: ["mcp_loki_*", "mcp_postgres_*"]
  
  "Create Migration":
    description: "Add database migration with database agent"
    scope: "v2/db_service/migrations/**"
    agent_hint: "Database_Agent"
    tools: ["mcp_postgres_*", "mcp_filesystem_*"]
    critical: "Only Database_Agent can create migrations"
```

## Monitoring Agent Activity

```yaml
Check_Agent_Status:
  - Review Cursor 2.0 multi-agent sidebar
  - Check GitHub Issues for coordination
  - Review TODO lists for task tracking
  - Use mcp_loki_* to monitor service logs from changes
  - Check git worktrees for active branches
```

---

## Quick Reference

**To spawn multiple agents:**
- Open Cursor 2.0 multi-agent sidebar
- Assign tasks with explicit path scopes
- Use agent role patterns above

**To prevent conflicts:**
- Use path scopes in prompts: `[Agent_Name scope: path/**]`
- Check GitHub Issues before starting
- Use exclude patterns for conflict zones

**To coordinate:**
- Create GitHub Issue for shared work
- Use TODO lists for task tracking
- Let git worktrees handle isolation

**Remember:**
- Up to 8 agents can run simultaneously
- Each agent uses isolated git worktree
- Path scoping prevents most conflicts
- Always coordinate database migrations (Database_Agent ONLY)

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
