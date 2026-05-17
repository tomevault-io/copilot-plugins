## claude-workflow

> Standard development workflow for Claude. Works everywhere: CLI, Desktop, IDE, and Cloud.

# claude-workflow

Standard development workflow for Claude. Works everywhere: CLI, Desktop, IDE, and Cloud.

On **CLI/Desktop**: Run `bash setup.sh` once after cloning to install plugins (superpowers, ECC, codex, etc.). Skills referenced below will activate automatically.

On **Claude Cloud**: No setup needed. This file IS the workflow — follow the instructions directly. Plugins, hooks, and MCP servers are declared in `.claude/settings.json` and `.mcp.json` (committed to the repo) and activate automatically on Cloud.

> **100% Cloud-ready**: MemPalace auto-installs on first session via SessionStart hook. All agents, skills, hooks, and MCP config are repo-committed. No local setup required.

---

## Standard Workflow

Follow this workflow for ALL implementation tasks. Each step is mandatory unless explicitly skipped by the user. If a skill mentioned below is available, invoke it. If not, follow the written instructions directly.

### Session Start (MANDATORY — run at the beginning of EVERY session)

Before any work begins, load project memory to establish continuity:

1. **Load MemPalace context** — if MemPalace MCP is available, run `wake-up` or search for this project's wing to load critical facts (~170 tokens).
2. **Check for open work** — search memory for: unfinished tasks, blockers from last session, pending decisions.
3. **Read recent git log** — `git log --oneline -10` to see what happened since last session.
4. **Read active plan** — check `docs/superpowers/plans/` for any in-progress plan and its current phase.
5. **Brief the user** — summarize what you found: where we left off, what's next, any open questions.

If MemPalace is not available, fall back to git log + plan docs + CLAUDE.md context only.

> This step ensures no context is lost between sessions. Skip ONLY if the user explicitly says "fresh start".

---

### Step 0: Research & Reuse

**Before writing ANY new code**, spawn the **researcher** agent or search manually:

1. Search GitHub for existing implementations (`gh search code`, `gh search repos`)
2. Check library docs for API behavior and patterns (Context7 or official docs)
3. Search package registries (npm, PyPI, crates.io) — prefer battle-tested libraries over hand-rolled code
4. Search the web for broader context only if steps 1-3 are insufficient
5. Look for open-source projects that solve 80%+ of the problem

Prefer adopting a proven approach over writing from scratch. **The researcher agent is ALWAYS the first to run** — no exceptions.

> CLI skill chain: `search-first` → `docs` → `deep-research` → `exa-search`

### Step 1: Brainstorm & Spec

**Mandatory before any creative or design work.** Do not jump to implementation.

1. Generate multiple approaches (at least 3). Consider trade-offs for each.
2. **Expand into a full spec** using `spec-expander` skill:
   - Interview the user to fill gaps (5-8 focused questions)
   - Write structured spec: Problem, Users, Requirements (MUST/SHOULD/COULD), Acceptance Criteria, Out of Scope
   - Save to `docs/specs/YYYY-MM-DD-[name].md`
3. **Review the spec** using `spec-reviewer` skill:
   - Score 7 dimensions (Clarity, Testability, Scope, Feasibility, Consistency, Security, Completeness)
   - Adversarial challenge on every MUST requirement
   - Verdict: KILL / FIX / SHIP — only proceed to planning on SHIP
4. If available, get a cross-model review via `/codex:review` on the spec file for an independent perspective.

> CLI skills: `superpowers:brainstorming` → `spec-expander` → `spec-reviewer`
> Optional plugin: `spec-kit` (athola/claude-night-market) for additional spec tooling

### Step 2: Plan

1. Create a structured implementation plan before writing code:
   - Break into phases (each independently testable)
   - Identify dependencies between phases
   - List risks and mitigation strategies
   - Estimate complexity per phase
2. Write the plan to `docs/superpowers/plans/YYYY-MM-DD-name.md`
3. Get user confirmation before proceeding to implementation

> CLI skills: `superpowers:writing-plans` + **planner** agent

### Step 3: Implement

1. Execute the plan phase by phase. Do not skip ahead.
2. For each phase, follow TDD strictly:
   - **RED**: Write the test first. Run it. It MUST fail.
   - **GREEN**: Write the minimal code to make the test pass.
   - **IMPROVE**: Refactor while keeping tests green.
3. Target 80%+ test coverage.
4. Use parallel agents/subagents for independent tasks within a phase.
5. After each file edit, review your own change for obvious issues before moving on.

> CLI skills: `superpowers:executing-plans` + `superpowers:subagent-driven-development` + `tdd` + `quality-gate`

### Step 4: Review

1. Review all code changes before considering work done:
   - **Correctness**: Does it do what was asked? Edge cases handled?
   - **Security**: No hardcoded secrets? Inputs validated? Queries parameterized? Auth checked?
   - **Quality**: Functions < 50 lines? Files < 800 lines? No deep nesting? Immutable patterns?
   - **Tests**: Do tests cover the happy path AND error cases?
2. For security-sensitive code (auth, payments, user input, DB, crypto): do an explicit security review.
3. For database changes: review schema design, query performance, migration safety.

> CLI skills: `superpowers:requesting-code-review` + `python-reviewer` / `typescript-reviewer` / `go-reviewer` / `rust-reviewer` + `security-reviewer` + `database-reviewer`

### Step 5: Verify

**Never say "done" without this step.**

1. Re-read the original request. Does the implementation match?
2. Run all tests. Do they pass?
3. Check for regressions — did anything else break?
4. Verify the user can actually use what was built (not just that it compiles).
5. List any known limitations or follow-up items.

> CLI skills: `superpowers:verification-before-completion` + `verify` + `context-budget`

### Step 6: Git

1. Stage only relevant files (never `git add .` blindly).
2. Use conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, `perf:`, `ci:`
3. Commit message: focus on WHY, not WHAT. The diff shows the what.
4. One logical change per commit.

> CLI skills: `superpowers:finishing-a-development-branch` + `superpowers:using-git-worktrees`

### Step 7: Learn

After completing a task, capture what was learned:

1. What patterns emerged that could be reused?
2. What was surprising or non-obvious?
3. What would you do differently next time?
4. Document decisions and their reasoning in commit messages or docs.

> CLI skills: `learn` / `learn-eval` → `instinct-status` → `promote` → `prune`

### Session End (MANDATORY — run before closing ANY session)

Before the session ends, persist everything learned for future sessions:

1. **Store decisions** — save all architectural decisions, trade-offs made, and their reasoning to MemPalace.
2. **Store current state** — what was completed, what's still open, what's blocked and why.
3. **Store patterns** — any reusable patterns, gotchas, or project-specific knowledge discovered.
4. **Store context for next session** — a brief "handoff note" so the next session can pick up immediately:
   - Current task and progress
   - Files actively being worked on
   - Open questions for the user
   - Next steps in priority order
5. **Verify memory consistency** — search MemPalace for contradictions with what was just stored. Update or flag conflicts.

If MemPalace is not available, write a `docs/session-log.md` entry instead.

> This step ensures the next session starts with full context. NEVER skip this step — the 2 minutes spent here save 20 minutes of re-discovery next session.

---

## Model-Routing

When spawning agents, choose the right model:

| Task type | Model | Why |
|-----------|-------|-----|
| Simple extraction, classification, formatting | **Haiku** | Fast, cheap ($0.80/M), 90% of Sonnet quality |
| Code generation, reviews, research, most work | **Sonnet** (default) | Best cost/quality for coding ($3/M) |
| Architecture, deep reasoning, root-cause analysis | **Opus** | Deepest reasoning ($15/M), use sparingly |

Default to Sonnet. Only escalate to Opus when reasoning depth matters. Drop to Haiku for mechanical tasks.

> **Advisor Strategy** (API/SDK only): Sonnet can auto-consult Opus mid-run via `advisor_20260301` tool — +2.7% accuracy, 11.9% cheaper than Opus alone. See [claude.com/blog/the-advisor-strategy](https://claude.com/blog/the-advisor-strategy).

## Design-Routing

For ANY UI/design task (components, pages, styling, colors, typography, charts, layouts):

1. **Brainstorm** — generate 3+ design approaches with trade-offs
2. **Design decisions** — choose layout, hierarchy, spacing, color, typography. Decide WHAT the design should be.
3. **Implementation** — translate design into code. Choose HOW: CSS approach, animation library, component structure.
4. **Review** — check accessibility, responsiveness, performance, consistency.

> CLI skill chain: `superpowers:brainstorming` → `frontend-design` → `ui-ux-pro-max` → `frontend-patterns`

## 3-Agent Harness (for complex features)

For features that span multiple files or require deep implementation:

1. **PLAN** → Spawn **planner** agent (`.claude/agents/planner.md`) to decompose into phases
2. **IMPLEMENT** → Spawn **implementer** agent (`.claude/agents/implementer.md`) — one phase at a time, fresh context per phase, commits progress to `docs/progress.md`
3. **EVALUATE** → Spawn **evaluator** agent (`.claude/agents/evaluator.md`) — validates against spec, runs tests, verdict per phase (PASS/WARN/FAIL)

Key principle: **Separate planning from execution from evaluation.** The planner can't be biased by implementation details. The evaluator provides "hard" feedback the implementer can't hallucinate away.

> Agents defined in `.claude/agents/`. Also available: **reviewer**, **researcher**, **security-reviewer**
> CLI skills: `/codex:rescue` + `/codex:review` for cross-model perspective

## Autonomous Execution (Ralph Loop)

For long-running tasks that require hours of autonomous work:

**The Pattern:** Instead of one long session, use iterative loops with fresh context per iteration. State is externalized to the filesystem (git commits, progress files, specs).

```
Session 1: Plan → commit plan + progress.md
Session 2: Read progress.md → implement phase 1 → commit → update progress.md
Session 3: Read progress.md → implement phase 2 → commit → update progress.md
...
Session N: Read progress.md → evaluate → final commit
```

**How to trigger:**
- **Cloud**: Use `/schedule` for cron-based autonomous execution on Anthropic infrastructure (runs even when your laptop is off)
- **CLI**: Use `claude -p` (headless mode) in a loop script
- **Remote**: Start with `--remote` — session persists after browser close

**Safeguards:**
- Commit after every meaningful change (recovery points)
- Cap iterations (2-3 for code gen, 10-20 for outer loop)
- Use test suites as primary feedback loop (not self-assessment)
- Write `docs/progress.md` after each phase so next iteration knows the state

> Inspired by [multi-agent-ralph-loop](https://github.com/alfredolopez80/multi-agent-ralph-loop): Ralph + MemPalace + Agent Teams + 4-stage quality gates

## Context & Memory Management

**MemPalace** is the persistent memory layer across all sessions. Session Start and Session End (see workflow above) are the mandatory checkpoints. Between them:

| Situation | What to do |
|-----------|-----------|
| Context filling up | Summarize completed work, drop verbose tool outputs, keep only what's needed for current task |
| Long session | At natural milestones, offer to summarize progress and compress context |
| Resuming previous work | Run **Session Start** protocol (git log + plan docs + MemPalace search) |
| Multi-file refactoring | Break into smaller commits. Don't hold too many files in context at once. |
| Mid-session decision | Store immediately to MemPalace — don't wait for Session End if the decision is important |

> CLI skills (ECC): `strategic-compact` (PreToolUse hook), `context-budget` (skill + command), `save-session`, `resume-session`
> Superpowers: SessionStart hook re-injects skills context on `startup|clear|compact` automatically
> Persistent memory: **MemPalace** MCP server — local-first, 96.6% LongMemEval, ~170 tokens always-on. Setup: `pip install mempalace && claude mcp add mempalace -- python -m mempalace.mcp_server`

## Continuous Learning

Every session generates knowledge. Capture it systematically:

| What to capture | Where to store | When |
|----------------|---------------|------|
| Architectural decisions + reasoning | MemPalace (decisions hall) | Immediately when decided |
| Reusable patterns discovered | MemPalace (patterns hall) | During Step 7 (Learn) |
| Anti-patterns / gotchas | MemPalace (anti-patterns hall) | When encountered |
| Project-specific fixes | MemPalace (fixes hall) | After debugging |
| Session handoff notes | MemPalace + `docs/progress.md` | Session End (mandatory) |

**Learned Rules Taxonomy** (from MemPalace):
- **Halls** (by type): decisions, patterns, anti-patterns, fixes
- **Rooms** (by topic): hooks, memory, agents, security, testing
- **Wings** (by scope): global (cross-project) and project-specific

**Quality filter:** ~46% of auto-learned rules are noise. Before storing, ask: *"Would this save time in a future session?"* If not, don't store it.

> CLI skills: `learn` / `learn-eval` → `instinct-status` → `promote` → `prune`

## Bei Problemen

| Situation | What to do |
|-----------|-----------|
| Bug or test failure | Read the error. Check assumptions. Reproduce minimally. Fix root cause, not symptoms. |
| Build breaks | Read the full error output. Fix incrementally. Verify after each fix. |
| Stuck on approach | Step back. Re-read the requirement. Try a fundamentally different approach. |
| Context too large | Summarize what's done, what's left. Start fresh section if needed. |
| Need second opinion | Explain the problem clearly, list what you've tried, ask for alternative approaches. |

> CLI skills: `superpowers:systematic-debugging`, `build-fix`, `codex:rescue`, `security-scan`

## Coding Standards

- **Immutability**: Create new objects, never mutate existing ones
- **Small files**: 200-400 lines typical, 800 max. Functions < 50 lines.
- **Error handling**: Explicit at every level. Never silently swallow errors.
- **Input validation**: Validate at system boundaries. Fail fast with clear messages.
- **Security**: No hardcoded secrets. Parameterized queries. Sanitized HTML. CSRF protection.
- **No over-engineering**: Don't add features, abstractions, or "improvements" beyond what was asked.

## Agent Orchestration

### Subagents (single tasks)

Use sub-agents proactively when the situation calls for it:

| Situation | Agent (`.claude/agents/`) |
|-----------|-----------|
| Complex feature with multiple phases | **planner** — decompose into phased plan |
| Implementation of a plan phase | **implementer** — execute one phase with TDD |
| Validate completed work against spec | **evaluator** — PASS/WARN/FAIL per phase |
| Code was just written or modified | **reviewer** — review for quality, security, bugs |
| Research before building | **researcher** — find existing solutions |
| Touching auth, payments, user data | **security-reviewer** — check for vulnerabilities |

Run independent agents in **parallel**. Don't wait for one to finish if another can start.

### Agent Teams (complex multi-agent workflows)

For tasks requiring 3+ parallel workstreams, use Agent Teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, enabled in `.claude/settings.json`):

- **3-5 teammates** for most workflows, 5-6 tasks per teammate
- Each teammate owns different files to avoid conflicts
- Shared task list with dependency tracking
- Teammates communicate directly via mailbox
- Use the **3-Agent Harness** pattern: planner → implementer → evaluator

**Best practices:**
- **Always plan before spawning** — the lead needs full scope first
- **Pre-allocate file ownership** — each teammate gets exclusive write access to specific modules
- **Read-heavy > write-heavy** — reviews, testing, analysis work best in parallel
- **Cap at 5 teammates** — beyond that, coordination costs exceed benefits
- **researcher runs first** — before any team spawns, research existing solutions
- Quality gates via hooks: `TaskCompleted` (test gate), `TeammateIdle` (prevent premature stop)

> Agent Teams work on both CLI and Cloud. Teammate definitions live in `.claude/agents/`.
> Hooks for team quality gates configured in `.claude/settings.json`.

---
> Source: [moeffel/claude-workflow](https://github.com/moeffel/claude-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
