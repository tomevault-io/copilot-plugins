## multi-agent-examples

> Practical examples for using multiple agents in Cursor 2.0


# 🤖 Multi-Agent Usage Examples

This file provides practical examples of how to use multiple agents simultaneously in your NEXUS v2 workspace.

> **💡 See [Custom Commands Proposal](mdc:../custom-commands-proposal.md) for 10 tailored commands specific to your NEXUS v2 stack**

## Quick Start: How to Use Multiple Agents

### Step 1: Open Multi-Agent Interface
- In Cursor 2.0, look for the **multi-agent sidebar** (new UI element)
- This sidebar shows all active agents and their status
- You can spawn new agents from this interface

### Step 2: Assign Tasks with Scope Hints

When prompting agents, include scope information to route to the right agent:

```
Example 1: Frontend Agent
"Improve the strategy configuration UI component [Frontend_Agent scope: v2/frontend/src/components/strategies/]"
```

```
Example 2: Backend Agent
"Add new REST endpoint for position history [Backend_Service_Agent scope: v2/api_gateway/routes/]"
```

```
Example 3: Database Agent
"Create migration for adding stop_loss column to positions table [Database_Agent scope: v2/db_service/migrations/]"
```

```
Example 4: Testing Agent
"Write integration tests for trade execution flow [Testing_Agent scope: v2/trade_engine/tests/]"
```

## Real-World Examples

### Example 1: Parallel Service Development

**Scenario**: You want to improve both signal_engine and trade_engine simultaneously.

**Setup**:
1. Create GitHub Issue #456: "Enhance signal processing and trade execution"
2. Break into sub-tasks:
   - Task A: Improve signal processing (signal_engine)
   - Task B: Optimize trade execution (trade_engine)

**Agent Prompts**:

```
Agent 1 (Backend_Service_Agent):
"Implement improved signal filtering algorithm [Backend_Service_Agent scope: v2/signal_engine/]"
```

```
Agent 2 (Backend_Service_Agent):
"Optimize trade execution latency [Backend_Service_Agent scope: v2/trade_engine/]"
```

**Why this works**:
- Different services = different file paths
- No file conflicts (git worktrees isolate)
- Both can run simultaneously

### Example 2: Full-Stack Feature Development

**Scenario**: Add new "Risk Dashboard" feature requiring frontend, backend, and database work.

**Setup**:
1. Create GitHub Issue #789: "Risk Dashboard Feature"
2. Break into coordinated tasks:
   - Database: Create risk_metrics table
   - Backend: Add risk calculation API
   - Frontend: Build dashboard UI
   - Testing: E2E tests

**Agent Prompts**:

```
Agent 1 (Database_Agent) - Run FIRST:
"Create migration for risk_metrics table with columns: strategy_id, var_value, max_drawdown [Database_Agent scope: v2/db_service/migrations/]"
```

```
Agent 2 (Backend_Service_Agent) - Run AFTER Agent 1:
"Implement risk calculation API endpoint that queries risk_metrics table [Backend_Service_Agent scope: v2/api_gateway/]"
```

```
Agent 3 (Frontend_Agent) - Run AFTER Agent 2:
"Build Risk Dashboard component that displays risk metrics from API [Frontend_Agent scope: v2/frontend/src/pages/]"
```

```
Agent 4 (Testing_Agent) - Run AFTER all:
"Write E2E tests for Risk Dashboard feature [Testing_Agent scope: v2/frontend/tests/]"
```

**Coordination**:
- Use GitHub Issue comments to track progress
- Agents wait for dependencies (database → backend → frontend)
- Final agent (Testing_Agent) validates integration

### Example 3: Observability Improvements

**Scenario**: Add Grafana dashboards and improve logging across services.

**Setup**:
1. Observability_Agent handles Grafana/Prometheus
2. Backend_Service_Agent adds structured logging

**Agent Prompts**:

```
Agent 1 (Observability_Agent):
"Create Grafana dashboard for trade execution metrics [Observability_Agent scope: v2/observability_stack/grafana/]"
```

```
Agent 2 (Backend_Service_Agent):
"Add structured logging to trade_engine with correlation IDs [Backend_Service_Agent scope: v2/trade_engine/]"
```

**Why this works**:
- Different paths (observability_stack vs service code)
- No conflicts
- Both improvements can be merged independently

### Example 4: Bug Fix + Feature Development

**Scenario**: Fix authentication bug while developing new feature.

**Agent Prompts**:

```
Agent 1 (Backend_Service_Agent) - Bug Fix:
"Fix JWT token expiration validation bug [Backend_Service_Agent scope: v2/auth_service/]"
```

```
Agent 2 (Frontend_Agent) - New Feature:
"Add strategy performance comparison chart [Frontend_Agent scope: v2/frontend/src/components/]"
```

**Why this works**:
- Different services (auth_service vs frontend)
- Different priorities (bug fix vs feature)
- Can run simultaneously without conflicts

## Conflict Avoidance Patterns

### ❌ BAD: Conflicting Agents

```
Agent 1: "Update shared utility function [v2/shared/utils/]"
Agent 2: "Refactor shared utility function [v2/shared/utils/]" 
```

**Problem**: Both agents modifying same files → conflicts

### ✅ GOOD: Coordinated Update

```
Step 1: Create GitHub Issue #999: "Refactor shared utilities"
Step 2: Assign to single agent OR coordinate via issue comments
Step 3: One agent makes changes, others wait
```

### ✅ GOOD: Non-Conflicting Parallel

```
Agent 1: "Update trade_engine to use shared utils [v2/trade_engine/]"
Agent 2: "Update signal_engine to use shared utils [v2/signal_engine/]"
```

**Why**: Different services, same shared dependency → no conflicts

## Using Custom Commands

In Cursor dashboard, you can create custom commands that automatically scope agents:

**Command Definition**:
```
Command Name: "Add Frontend Component"
Description: "Create new React component"
Scope Pattern: "v2/frontend/src/components/**"
Agent Hint: Frontend_Agent
Tool Preferences: Browser Agent, mcp_filesystem_*
```

**Usage**:
- Trigger command from Cursor
- Command automatically routes to Frontend_Agent
- Path scope is pre-configured

## Monitoring Multi-Agent Work

### Check Active Agents
- Open multi-agent sidebar in Cursor 2.0
- See all active agents and their current tasks
- Monitor progress in real-time

### Validate No Conflicts
```bash
# Check git worktrees
git worktree list

# Check for unmerged branches
git branch --no-merged

# Review agent changes
git log --all --oneline --graph
```

### Use Loki MCP for Validation
```yaml
After agents make changes:
  - Query service logs: mcp_loki_query("{service='trade_engine'} |= 'ERROR'")
  - Validate no regressions introduced
  - Check performance metrics
```

## Best Practices Summary

### ✅ DO
- Use explicit path scopes in prompts
- Create GitHub Issues for coordination
- Run independent services in parallel
- Test agent changes independently
- Use TODO lists for tracking

### ❌ DON'T
- Have multiple agents modify same files
- Skip path scoping
- Create migrations outside Database_Agent
- Modify shared libraries without coordination
- Forget to validate with Browser Agent (frontend)

## Troubleshooting

### Issue: Agents modifying same files
**Solution**: Check agent scopes, use exclude patterns, coordinate via GitHub Issue

### Issue: Confusion about which agent to use
**Solution**: Reference [multi-agent-coordination.mdc](mdc:.cursor/rules/multi-agent-coordination.mdc) for role definitions

### Issue: Conflicts during merge
**Solution**: Standard git conflict resolution, both agents should validate after merge

### Issue: Agent not respecting scope
**Solution**: Make path scope more explicit in prompt, use exclude patterns in rule file

---

## Quick Reference Commands

**Spawn agent for frontend:**
```
"Add new React component [Frontend_Agent scope: v2/frontend/]"
```

**Spawn agent for backend:**
```
"Implement new service endpoint [Backend_Service_Agent scope: v2/{service_name}/]"
```

**Spawn agent for database:**
```
"Create migration [Database_Agent scope: v2/db_service/migrations/]"
```

**Spawn agent for testing:**
```
"Write tests [Testing_Agent scope: **/tests/**]"
```

**Spawn agent for observability:**
```
"Add Grafana dashboard [Observability_Agent scope: v2/observability_stack/]"
```

Remember: Include `[Agent_Name scope: path/**]` in your prompts to route tasks correctly!

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
