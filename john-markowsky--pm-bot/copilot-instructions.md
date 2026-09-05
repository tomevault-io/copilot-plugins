## sprint-planning

> Atomic task decomposition for complex projects using Dillon Mulroy's methodology


# 🎯 Sprint Planning Framework

**Break complex projects into atomic tasks. Every sprint = demoable software increment.**

## Core Rules

### Task Atomicity
- **Single Responsibility**: One clear, focused objective
- **Commitable Unit**: Complete implementation in one commit
- **Testable**: Includes validation method (tests/Browser Agent/docs)
- **Independent**: Can be implemented without blocking other tasks

### Sprint Requirements
- **Demoable Deliverable**: Runnable, testable software increment
- **Buildable Foundation**: Next sprint can build on current work
- **Measurable Goal**: Clear success criteria
- **Realistic Scope**: 2-4 week delivery cycle

### Validation Standards
```yaml
# Every task must specify validation method:
✅ Database: "psql schema validation"
✅ Backend: "pytest unit/integration tests"
✅ Frontend: "Browser Agent E2E (zero console errors)"
✅ Integration: "Loki log analysis + MCP validation"
✅ Documentation: "Updated existing docs (no new files)"
```

## Integration with Existing Workflow

### PLAN Mode Enhancement
```yaml
# For complex projects (>5 tasks), use sprint breakdown:
1. Analyze with MCP tools
2. Create sprint breakdown (/sprint-breakdown command)
3. Generate atomic task list
4. Get user approval
5. Execute via ACT mode
```

### ACT Mode Execution
```yaml
# Execute sprints sequentially:
Sprint 1 → Demo → Sprint 2 → Demo → Sprint 3 → Final Demo

# Never parallel sprints without explicit approval
# Each sprint must be 100% complete before next
```

### Multi-Agent Coordination
```yaml
# Route tasks to appropriate agents:
Database_Agent: "v2/db_service/migrations/**" (exclusive)
Backend_Service_Agent: "v2/*_service/**"
Frontend_Agent: "v2/frontend/**"
Documentation_Agent: "v2/docs/**" (update existing only)
Observability_Agent: "v2/observability_stack/**"
```

## Task Template

### Required Format
```yaml
**Task Name**
Scope: [Service/Agent assignment]
Validation: [Specific validation method]
Dependencies: [What must complete first]
Estimated: [Realistic time estimate]
```

### Example Tasks
```yaml
**Database Schema Migration**
Scope: [Database_Agent: v2/db_service/migrations/]
Validation: psql schema check + test data insertion
Dependencies: None
Estimated: 2 hours

**API Endpoint Implementation**
Scope: [Backend_Service_Agent: v2/api_gateway/]
Validation: pytest integration tests + API documentation
Dependencies: Database migration complete
Estimated: 4 hours

**React Component Development**
Scope: [Frontend_Agent: v2/frontend/src/components/]
Validation: Browser Agent E2E (zero console errors)
Dependencies: API endpoint available
Estimated: 6 hours
```

## Sprint Planning Process

### 1. Project Analysis
```yaml
# Use MCP tools to understand scope:
mcp_repo-indexer_get_dependency_graph({ max_depth: 3 })
mcp_docs-indexer_get_api_endpoints_catalog()
mcp_config-indexer_search_configuration({ query: "service dependencies" })
```

### 2. Sprint Decomposition
```yaml
# Break into 2-4 week increments:
Sprint 1: Foundation (data layer, basic APIs)
Sprint 2: Core Features (business logic, UI)
Sprint 3: Integration (end-to-end workflows)
Sprint 4: Polish (performance, error handling)
```

### 3. Task Granularity
```yaml
# Each task should be:
- 2-8 hours of focused work
- Clear acceptance criteria
- Independent of other tasks
- Demoable when complete
```

### 4. Risk Assessment
```yaml
# Identify and mitigate:
- External dependencies (APIs, services)
- Complex integrations (WebSocket, Kafka)
- Performance requirements
- Security considerations
```

## Success Metrics

### Task Completion
- ✅ Code committed with passing validation
- ✅ Tests green (pytest/Browser Agent)
- ✅ No TODO comments in production code
- ✅ Documentation updated (existing files only)

### Sprint Completion
- ✅ All tasks complete and committed
- ✅ Demoable software increment
- ✅ Can be built upon by next sprint
- ✅ Zero critical bugs or console errors

### Project Completion
- ✅ All sprints delivered
- ✅ End-to-end functionality working
- ✅ Production deployment ready
- ✅ Performance and security validated

## Common Pitfalls

### ❌ Too Large Tasks
```yaml
WRONG: "Build complete user authentication system"
RIGHT: "Implement JWT token validation" + "Create login API endpoint" + "Add password hashing"
```

### ❌ Missing Validation
```yaml
WRONG: "Add error handling"
RIGHT: "Add error handling with comprehensive test coverage"
```

### ❌ Non-Demoable Sprints
```yaml
WRONG: "Design system architecture"
RIGHT: "Implement core API with working endpoints"
```

### ❌ Parallel Sprint Execution
```yaml
WRONG: Working on Sprint 1 and Sprint 2 simultaneously
RIGHT: Complete Sprint 1 demo before starting Sprint 2
```

## Integration Points

### Git Workflow
- **One commit per atomic task** (when validation passes)
- **Single commit per sprint completion** (when demo ready)
- **Never commit partial work or failing tests**

### Documentation Standards
- **Update existing docs** (never create new .md files)
- **Document during implementation** (not as separate tasks)
- **Use existing issue tracking** (GitHub Issues > new docs)

### Testing Requirements
- **TDD for backend**: RED → GREEN → REFACTOR
- **Browser Agent for frontend**: Iterative until zero console errors
- **MCP validation**: Logs, database, configuration checks
- **Integration testing**: End-to-end workflows

---

**Applied To**: Complex projects requiring sprint breakdown
**Complements**: PLAN/ACT mode, multi-agent coordination, TDD workflow
**Validation**: MCP servers + Browser Agent required for all frontend tasks

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
