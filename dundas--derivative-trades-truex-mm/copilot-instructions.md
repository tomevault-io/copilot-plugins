## derivative-trades-truex-mm

> Every session should move the system closer to production-ready. The full checklist lives in `memory/production-readiness.md`. Priority order:

# true_markets_mm

## Primary Goal: Production Readiness

Every session should move the system closer to production-ready. The full checklist lives in `memory/production-readiness.md`. Priority order:

1. **Security** — API auth, input validation, CORS lockdown, rate limiting
2. **Reliability** — Graceful shutdown, process supervision, connection resilience
3. **Observability** — Structured logging, metrics, health monitoring, alerting
4. **Testing** — E2E API tests, integration tests, load tests, chaos tests
5. **Operations** — Production Docker, DB migrations, backup, deployment runbook

When starting a session, check `memory/production-readiness.md` for the next unchecked item and propose working on it. When finishing work, update the checklist.

## Autonomous Memory System

This project uses the agentbootup self-improvement system for continuous learning and autonomous operation.

### Memory Files (Always Consult)

**At session start, read these files:**

| File | Purpose |
|------|---------|
| `memory/MEMORY.md` | Core operational knowledge |
| `memory/daily/<today>.md` | Today's session log |
| `.ai/protocols/STANDARD_DEV_WORKFLOW.md` | 7-step code change pipeline |
| `.ai/protocols/AUTONOMOUS_OPERATION.md` | Decision authority, communication style |

**At session end, update**:
1. `memory/daily/<today>.md` - Session summary, decisions, learnings
2. `memory/MEMORY.md` - New permanent patterns (if discovered)

### Autonomous Operation Protocols

See `.ai/protocols/AUTONOMOUS_OPERATION.md` for complete protocols including:
- Decision-making authority (what to act on vs ask about)
- Phase gate protocol (when to pause for confirmation)
- Error handling protocol (fix immediately, never defer)
- Skill acquisition protocol (building permanent capabilities)
- Memory management protocol (what/when/how to update)

### Key Principles

**Decision-Making**:
- ✅ Act autonomously on: technical choices, testing, documentation, memory updates
- ❌ Ask for input on: destructive actions, external communications, strategic direction

**Communication Style**:
- Be decisive, not deferential
- State decisions with reasoning
- Signal confidence levels
- Silence = normal operation

**Error Handling**:
- Fix issues immediately
- Never mark tasks complete with caveats
- Test until it actually works
- Update memory with lessons learned

**Phase Gates**:
- Complete each phase fully
- Pause at major transitions
- Wait for explicit "Go" or "yes"
- No partial work left behind

### Skills System

**Location**: `.ai/skills/` (CLI-agnostic) or `.claude/skills/` (Claude-specific)

**Core Skills**:
- `skill-acquisition/` - Systematic skill building workflow
- `memory-manager/` - Automated memory management

**Creating New Skills**:
1. **Phase 0**: Check existing skills first (MANDATORY)
2. Only build if no existing skill covers the capability
3. Use skill-acquisition workflow for structured creation

### Task Management

**Use Claude Code native tasks**:
- TaskCreate - Create new tasks
- TaskUpdate - Update task status
- TaskList - View all tasks
- TaskGet - Get task details

**Coordinate with memory**:
- Tasks = tactical execution
- WORKQUEUE.md = strategic direction (if used)
- Memory = long-term knowledge

### Standing Orders

Execute continuously without being asked:

1. Check memory at session start
2. Monitor system health proactively
3. Learn continuously - update memory after significant interactions
4. Build skills permanently for novel challenges (check existing first!)
5. Pause at phase gates
6. Test before completion
7. Act proactively on routine items
8. Ask before destructive actions
9. Document decisions in daily logs
10. Fix issues immediately
11. **(SO-6)** Run `/pre-push-review` before every `git push` — including fix commits mid-review-loop
12. **(SO-7)** `pr-review-loop` Phase 0 delegates to `/pre-push-review` (not an inline semgrep/roborev block)
13. **(SO-8)** Smoke test runs after `/pre-push-review` passes, before push. Skip requires explicit category + reason in PR description. Valid skip categories: `DOCS_ONLY`, `CONFIG_ONLY`, `NO_SERVER_SURFACE`, `SMOKE_MISSING (must name follow-up PR)`
14. **(SO-13)** `/docs-generator` mandatory after PRs touching `scripts/`, `docs/`, or content pipeline files
15. **(SO-14)** If no review comments after 5 min on a PR, tag `@coderabbitai review`. Never merge on CI-only.

## Development Protocol (mandatory for ALL code changes)

**Canonical source: `.ai/protocols/STANDARD_DEV_WORKFLOW.md`**

Every code change follows this 7-step pipeline. No shortcuts.

1. **Skill-first** — enumerate skills before coding
2. **Dialectical coder** — `/dialectical-autocoder` for non-trivial changes (>3 files)
3. **Feature branch** — never commit to `main` directly (SO-11)
4. **Gate chain** (in order):
   - `4a` `/pre-push-review` — semgrep + roborev (SO-6)
   - `4b` Smoke test (SO-8). Skip categories: `DOCS_ONLY`, `CONFIG_ONLY`, `NO_SERVER_SURFACE`, `SMOKE_MISSING`
   - `4c` `/adversarial-reviewer` — must invoke via Skill tool (SO-9)
   - `4d` `/pr-review-loop` — tag `@coderabbitai review` if no comments after 5 min (SO-14)
5. **`/docs-generator`** — mandatory after PRs touching `scripts/`, `docs/`, or content pipeline (SO-13)
6. **Docs PR review** — `/pr-review-loop` on the docs PR
7. **Test locally after merge** — pull main, verify end-to-end

**Never merge on CI-only.** At least one code review (automated or human) must complete.

- **Auto-merge criteria**: 0 blocking issues + CI green — formal APPROVED not required
- **Agent PRs**: Include `@coderabbitai ignore` in PR body at creation to suppress bot noise

---
> Source: [dundas/derivative-trades-truex-mm](https://github.com/dundas/derivative-trades-truex-mm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
