## claude-code-agentic-coding-google-developers-group

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md Template

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

<!-- CUSTOMIZE: Replace with your project description -->
This is **[Your Project Name]**—[brief description of what the project does and its purpose].

**Production URLs:**
- Frontend: [your-frontend-url]
- Backend: [your-backend-url]

**Current Status:** [Describe current state: deployed, in development, MVP, etc. List key features/capabilities.]

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in PLANS.md) from design to implementation. Keep EXECPLAN.md updated as it is the source of truth and living document for this project.

## Living Documentation

EXECPLAN.md is the single source of truth for this project. You MUST:

1. **Read EXECPLAN.md at session start** - Always read it first to understand current state
2. **Update Progress section** - After completing any task, add a timestamped entry
3. **Update Surprises and Discoveries** - Document any bugs, unexpected behaviors, or workarounds found
4. **Update Decision Log** - Record any significant implementation decisions with rationale
5. **Update Outcomes and Retrospective** - At major stopping points, summarize what's working, what's not, and lessons learned

Never use temporary todo lists as the primary tracking mechanism. All progress must be recorded in EXECPLAN.md so the next session (or a new conversation) can pick up exactly where we left off.

When resuming work:
1. Read EXECPLAN.md first
2. Check the Progress section for incomplete milestones
3. Check Surprises and Discoveries for known issues
4. Continue from documented stopping point

### Source of Truth

<!-- CUSTOMIZE: List your key documentation files -->
All specifications and design decisions are documented in:
- `VISION.md` — Core requirements, design philosophy, success criteria
- `EXECPLAN.md` — Active milestones, known issues, progress tracking
- `.claude/memory/` — External memory (archived milestones, decisions, schemas, reference)


## Memory System

This project uses a tiered memory system to manage complexity.

### Core Memory (Always Loaded)
- `EXECPLAN.md`: Active state, current milestones, known issues
- `CLAUDE.md`: This file - instructions, conventions, memory access

### External Memory (Read on Demand)
Stored in `.claude/memory/`:

<!-- CUSTOMIZE: Replace with your project's memory files -->
| File | Contains | Read When |
|------|----------|-----------|
| `INDEX.md` | Summary of all memory files | Starting new topic |
| `milestones/[feature-area].md` | Completed milestone details | Debugging that feature |
| `decisions/architecture.md` | Structural choices | Proposing changes |
| `decisions/technology.md` | Stack choices | Evaluating alternatives |
| `decisions/patterns.md` | Implementation patterns | Adding features |
| `schemas/database.md` | Database schema | Database work |
| `schemas/api.md` | API specs | API modifications |
| `schemas/components.md` | UI components | UI work |
| `reference/context.md` | Glossary, structure | Onboarding |
| `reference/retrospective.md` | What worked, lessons | Planning |
| `quality/testing.md` | Test infrastructure, patterns, coverage | Writing tests |

### Starting New Work Protocol

Before implementing a new milestone, PROACTIVELY read relevant memory:

1. **Always read first:**
   - `.claude/memory/INDEX.md` - orient to available memory
   - `decisions/architecture.md` - check architectural constraints
   - `decisions/technology.md` - verify stack choices still apply

2. **Read based on work type:**
   <!-- CUSTOMIZE: Map your feature areas to memory files -->
   - [Feature Area 1] work → `milestones/[feature-area-1].md`
   - [Feature Area 2] work → `milestones/[feature-area-2].md`
   - Database changes → `schemas/database.md`
   - API changes → `schemas/api.md`
   - Testing work → `quality/testing.md`

3. **Check for patterns:**
   - Similar past milestones for implementation patterns
   - Related decisions for constraints and trade-offs
   - Known issues that may affect new work

4. **ALWAYS spawn 3 scout subagents before implementing:**

   **This is mandatory for EVERY milestone.** Spawn 3 parallel subagents to search archived memory before writing any code. Failing to understand the memory system will break the repo.

   | Agent | Focus | Searches | Returns |
   |-------|-------|----------|---------|
   | **Decisions Scout** | Constraints & trade-offs | `decisions/*.md` | Relevant constraints, past rationale |
   | **Patterns Scout** | Implementation precedents | `milestones/*.md` | Similar past work, patterns, gotchas |
   | **Schema Scout** | Data structures & APIs | `schemas/*.md`, `reference/*.md` | Affected tables, endpoints, types |

   **Do NOT skip this step.** The scouts ensure you understand existing patterns before making changes.

### Reactive Retrieval Triggers

Also read external memory when:
1. User asks about completed features
2. You encounter unexpected behavior
3. You're unsure about prior decisions
4. Debugging requires implementation context

### Memory Update Protocol

**This is mandatory for EVERY milestone.**

After completing work:
1. Archive detailed notes to appropriate memory file
2. Update EXECPLAN.md Progress (keep concise - link to archive)
3. Update INDEX.md if new file created
4. Add new decisions to decisions/*.md with rationale

**Writing style for memory:** Be concise. Sacrifice grammar for brevity but explain the system thoroughly and preserve full meaning. Use tables, bullet points, code snippets over prose. Link to source files with line numbers. Future sessions need context, not narrative.


## Technology Stack

<!-- CUSTOMIZE: Replace with your actual stack -->
| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | [Framework] | [Additional libraries] |
| Backend | [Language/Framework] | [Additional tools] |
| Database | [Database] | [Features used] |
| Hosting | [Hosting providers] | [Deployment method] |
| [Other layers] | [Technology] | [Notes] |


## Infrastructure

For deployment and operational procedures, see **`INFRA.md`**.

### Deployment Overview

<!-- CUSTOMIZE: Replace with your deployment setup -->
| Component | Host | Deploy Method |
|-----------|------|---------------|
| Frontend | [Host] | [Method] |
| Backend | [Host] | [Method] |
| Database | [Host] | [Method] |

### Before Deploying

<!-- CUSTOMIZE: Your pre-deployment checklist -->
1. Run build command to verify production build
2. Run lint to catch lint errors
3. Check `INFRA.md` for environment-specific requirements
4. Verify all environment variables are set

### Key Infrastructure Files

<!-- CUSTOMIZE: List your infrastructure files -->
- `INFRA.md` — Full deployment documentation, credentials reference, runbooks
- [Backend Dockerfile path] — Backend container definition
- [Docker Compose path] — Container orchestration
- [Frontend deploy config] — Frontend deployment configuration

### Common Operations

<!-- CUSTOMIZE: Your common operational commands -->
| Task | Command/Location |
|------|------------------|
| Deploy backend | See INFRA.md |
| View production logs | [Your log command] |
| Check health | [Your health check endpoint] |
| Rollback backend | See INFRA.md |
| Rollback frontend | [Your rollback method] |

### Infrastructure Changes

When making infrastructure changes:
1. Update `INFRA.md` first (documentation-first)
2. Test in development environment
3. Deploy to production
4. Verify with post-deployment checks

Never modify production infrastructure without:
- Documented rollback procedure
- Access to monitoring/logs
- Understanding of `INFRA.md` content

## Development Commands

<!-- CUSTOMIZE: Replace with your actual commands -->
```bash
# Frontend
cd [frontend-dir]/
npm install              # Install dependencies
npm run dev              # Development server
npm run build            # Production build
npm run lint             # Lint code
npm run test             # Run tests

# Backend
cd [backend-dir]/
[package manager install command]
[dev server command]
[test command]
```

## Key Architectural Decisions

<!-- CUSTOMIZE: List your key architectural decisions -->
- **[Decision 1]** ([rationale])
- **[Decision 2]** ([rationale])
- **[Decision 3]** ([rationale])

## Design Philosophy

<!-- CUSTOMIZE: Define your guiding principles -->
These principles guide all implementation decisions:

1. **[Principle 1]** — [Description]:
   - [Sub-point]
   - [Sub-point]

2. **[Principle 2]** — [Description]:
   - [Sub-point]
   - [Sub-point]

3. **[Principle 3]** — [Description]:
   - [Sub-point]
   - [Sub-point]

## Core Capabilities

<!-- CUSTOMIZE: List your application's main features -->
1. **[Capability 1]** — [Description]
2. **[Capability 2]** — [Description]
3. **[Capability 3]** — [Description]

## Testing Instructions

**TDD is mandatory for EVERY milestone.** No milestone is complete without passing tests. Use two test types: **Logic Tests** (automated) and **UI Tests** (browser automation).

### Agent Separation Policy (MANDATORY)

**The agent that writes tests MUST be different from the agent that implements the solution.** This prevents bias. For every milestone:
1. Spawn **Test Agent** first to write failing tests based on requirements
2. Spawn **Implementation Agent** to write code that passes the tests
3. Test Agent reviews coverage and adds edge cases

### Real Data Policy (MANDATORY)

**NEVER use mock data when real services are available.** This applies to EVERYTHING - databases, APIs, storage, auth, any service.

If a real endpoint exists, use it. If a real database exists, query it. If a real service exists, call it.

**Only mock when absolutely necessary:**
- External paid APIs to avoid costs during CI
- Time-sensitive operations (use fixed timestamps)
- Network failure scenarios (to test error handling)
- Third-party services with no test environment

### Logic Tests (Automated)

<!-- CUSTOMIZE: Your test tools and locations -->
| Tool | Location | Run Command |
|------|----------|-------------|
| [Backend test tool] | [test location] | [run command] |
| [Frontend test tool] | [test location] | [run command] |
| [E2E test tool] | [test location] | [run command] |

**What to test with Logic Tests:**
<!-- CUSTOMIZE: Your test targets -->
- Backend: API endpoints, business logic, validation, data transformations
- Frontend: Hook logic, utility functions, state management, form validation

**Alternative: MCP Tools for Testing**
The system has access to MCP browser automation tools (Chrome extension). If these are faster or more advantageous for a specific test scenario, use them instead of or alongside traditional test frameworks.

**Run before every commit:**
```bash
# CUSTOMIZE: Your test commands
[backend test command]
[frontend test command]
```

### UI Tests (Browser Automation)

Use Chrome extension for visual verification and user flow testing.

**What to test with UI Tests:**
- Visual rendering: Does the page look correct?
- User flows: [your key user flows]
- Component interactions: buttons, forms, modals
- Responsive design: 375px, 768px, 1024px, 1440px viewports

**UI Test Protocol:**
1. Start dev server
2. Navigate to localhost
3. Take screenshots of affected pages
4. Test user interactions (click, type, submit)
5. Test at mobile viewport (375px width)

### Pre-Commit Checklist

<!-- CUSTOMIZE: Your pre-commit checks -->
- [ ] Lint passes
- [ ] Build succeeds
- [ ] Backend tests pass
- [ ] Frontend tests pass
- [ ] UI visually verified for any UI changes

## Git Workflow

- Branch naming: `feature/<component>` or `fix/<issue>`
- Commit messages: Use conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`)
- Commit after completing each milestone in an ExecPlan

## Coding Constraints

**Do:**
<!-- CUSTOMIZE: Your positive constraints -->
- Follow the design philosophy above
- Handle errors gracefully
- Write tests BEFORE implementing new features (TDD)
- Run tests before committing
- Add test coverage for new endpoints and hooks
- Use UI tests for visual changes and user flow verification

**Do Not:**
<!-- CUSTOMIZE: Your negative constraints -->
- Store secrets in code or documentation files
- Skip tests when implementing new features
- Mark milestones complete without passing tests
- Use mock data when real services are available
- Commit API keys to git (use .env files)

## Success Metrics

<!-- CUSTOMIZE: How you measure success -->
The system succeeds if:
1. [Metric 1]
2. [Metric 2]
3. [Metric 3]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g-troiani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
