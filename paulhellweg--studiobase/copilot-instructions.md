## studiobase

> > This file is the **non-negotiable constitution** for every session, every agent, every task.

# CLAUDE.md — Dark Factory v6.1
# Autonomous SWE Lifecycle Pipeline

> This file is the **non-negotiable constitution** for every session, every agent, every task.
> Read this fully before doing ANYTHING.

---

## 🏭 What Is This

Dark Factory (DF) is a Spec-First autonomous software engineering pipeline. A structured Markdown spec
flows through specialized agents to produce deployed, tested, secure software — with zero hand-holding.

**Core Contract:** You receive a spec. You deliver working software. No excuses.

---

## 📋 Session Start Protocol (MANDATORY)

On EVERY session start, in this exact order:

1. Read `project-context.md` (stack, frameworks, conventions)
2. Read `tasks/task_plan.md` (current phase + open tasks)
3. Read `tasks/progress.md` (last 5 completed items)
4. **Read `tasks/lessons.md`** (errors you made before — STUDY these)
5. Check `tasks/findings.md` for unresolved blockers

Only after all 5 are read: proceed with the current task.

---

## 🔄 Pipeline Phases

```
SPEC → ARCHITECTURE → PLAN → IMPLEMENT → TEST → REVIEW → SECURITY → DEPLOY
```

Each phase gates the next. Never skip. Never go backwards without re-planning.

**Phase files:**
- `SPEC.md` — functional requirements, data models, user stories
- `ARCHITECTURE.md` — tech stack, DB schema, auth, folder structure
- `tasks/task_plan.md` — current sprint tasks (checkable items)
- `tasks/findings.md` — research outputs, blockers, decisions
- `tasks/progress.md` — what was completed and when
- `tasks/lessons.md` — **self-improvement log** (see §Self-Improvement Loop)

---

## 🧠 Self-Improvement Loop (NEW in v6.1)

After ANY correction from a user or failed verification:

1. Stop immediately
2. Identify the root cause (not the symptom)
3. Append to `tasks/lessons.md`:
   ```
   ## [YYYY-MM-DD] [Category]
   **Mistake:** What went wrong (specific, not vague)
   **Root Cause:** Why it happened
   **Rule:** The concrete rule that prevents recurrence
   **Pattern:** What to watch for next time
   ```
4. Apply the new rule immediately to the current task
5. At session start, scan lessons.md for patterns relevant to current work

Categories: `SECURITY | ARCHITECTURE | TESTING | PLANNING | ELEGANCE | TOOLING`

---

## 🎯 Workflow Orchestration

### 1. Plan Node — Always First
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write plan to `tasks/task_plan.md` with checkable items BEFORE coding
- Use `tasks/findings.md` for research outputs during planning
- If something goes sideways: STOP, update plan, then continue

### 2. Subagent Strategy
- Use subagents for parallel work, exploration, and research
- Keep main context window clean — offload to subagents liberally
- One focused task per subagent
- For complex problems: throw compute at it via subagents

### 3. Verification Before Done
- Never mark a task complete without proving it works
- Run tests, check logs, demonstrate correctness
- Ask: "Would a senior engineer approve this PR?"
- Diff behavior between main and your changes when relevant

### 4. Demand Elegance
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: **"Knowing everything I know now, implement the elegant solution"**
- Skip for simple obvious fixes — don't over-engineer
- Challenge your own implementation before calling it done

### 5. Autonomous Bug Fixing
- When given a bug report: fix it. No hand-holding required.
- Point at logs, errors, failing tests — then resolve them autonomously
- Zero context switching required from the user
- Go fix failing CI tests without being told how

---

## 🤖 Smart Model Routing

| Agent | Model | Reason |
|-------|-------|--------|
| security-auditor | opus | Highest stakes, needs deep reasoning |
| architect-reviewer | opus | Architecture quality gates |
| All code agents | sonnet | Best cost/quality for implementation |
| devops-engineer | haiku | Simple, repetitive infra tasks |
| spec-writer | sonnet | Good enough, fast |

---

## 🔒 Security Constitution (Non-Negotiable)

### Hard Rules — Violation = Pipeline HALT
- **Never** commit secrets, API keys, tokens to any file
- **Never** store PII in plaintext — AES-256 minimum
- **Never** use `eval()`, `dangerouslySetInnerHTML` without explicit sanitization
- **Never** trust user input — validate + sanitize at every boundary
- **Never** use auto-increment IDs in public APIs — UUIDs only
- **Never** log sensitive data (passwords, tokens, PII)
- **Always** use parameterized queries — no string concatenation in SQL
- **Always** use `httpOnly` cookies for auth tokens
- **Always** rate limit auth endpoints

### Security Agent Has VETO Power
The `security-auditor` agent can block any deployment.
A security veto cannot be overridden by any other agent.
Only a human can override a security veto with explicit confirmation.

### OWASP Top 10 — Always Check
A1: Broken Access Control | A2: Cryptographic Failures | A3: Injection
A4: Insecure Design | A5: Security Misconfiguration | A6: Vulnerable Components
A7: Auth Failures | A8: Data Integrity Failures | A9: Logging Failures | A10: SSRF

---

## 📦 Task Management

1. **Plan First** — Write `tasks/task_plan.md` with checkable items
2. **Verify Plan** — Check it before starting implementation
3. **Track Progress** — Mark items complete as you go, update `tasks/progress.md`
4. **Explain Changes** — High-level summary at each step
5. **Document Results** — Add review section to `tasks/task_plan.md` when done
6. **Capture Lessons** — Update `tasks/lessons.md` after corrections (see §Self-Improvement Loop)

---

## ⚙️ Core Principles

- **Simplicity First** — Make every change as simple as possible. Impact minimal code.
- **No Laziness** — Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact** — Changes touch only what's necessary. Avoid introducing bugs.
- **Compaction Survivability** — All state lives in files, never only in context.
- **No Checkpoints** — Pipeline runs autonomously from start to finish.

---

## 🚀 Commands

| Command | Action |
|---------|--------|
| `/df:start` | Begin pipeline from requirements.md |
| `/df:status` | Show current phase + open tasks |
| `/df:resume` | Resume after context reset (reads all state files) |
| `/df:elegance` | Trigger elegance review on current implementation |
| `/df:lessons` | Show lessons.md summary for current project |

---
> Source: [PaulHellweg/studiobase](https://github.com/PaulHellweg/studiobase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
