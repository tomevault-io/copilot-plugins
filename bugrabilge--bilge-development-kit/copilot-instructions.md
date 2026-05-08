## bilge-development-kit

> > This file defines how Claude Code behaves in this workspace.

# Bilge Development Kit - Claude Code Rules

> This file defines how Claude Code behaves in this workspace.
> Adapted from GEMINI.md for Claude Code's native tooling.

---

## MEMORY BANK (Session Start)

**At the start of every session**, check if `.agent/.claude/memory/` directory exists and read any available memory files:
- `MEMORY-activeContext.md` — current work state, recent sessions
- `MEMORY-patterns.md` — established code patterns and conventions
- `MEMORY-decisions.md` — architecture decision records
- `MEMORY-troubleshooting.md` — solved issues and lessons learned

Use this context to avoid re-discovering information. If memory files don't exist, suggest running `/remember` to initialize the Memory Bank.

---

## CRITICAL: AGENT & SKILL PROTOCOL (START HERE)

**MANDATORY:** You MUST read the appropriate agent file and its skills BEFORE performing any implementation.

### 1. Modular Skill Loading Protocol

```
Agent activated -> Check frontmatter "skills:" -> Read SKILL.md (INDEX) -> Read specific sections
```

- **Selective Reading:** DO NOT read ALL files in a skill folder. Read `SKILL.md` first, then only read sections matching the user's request.
- **Rule Priority:** P0 (CLAUDE.md root) > P1 (Agent .md) > P2 (SKILL.md). All rules are binding.

### 2. Enforcement Protocol

1. **When agent is activated:**
   - Read Rules -> Check Frontmatter -> Load SKILL.md -> Apply All.
2. **Forbidden:** Never skip reading agent rules or skill instructions. "Read -> Understand -> Apply" is mandatory.

---

## REQUEST CLASSIFIER (STEP 1)

**Before ANY action, classify the request:**

| Request Type     | Trigger Keywords                           | Active Tiers                         | Result                      |
| ---------------- | ------------------------------------------ | ------------------------------------ | --------------------------- |
| **QUESTION**     | "what is", "how does", "explain"           | TIER 0 only                          | Text Response               |
| **SURVEY/INTEL** | "analyze", "list files", "overview"        | TIER 0 + Explore subagent            | Session Intel (No File)     |
| **SIMPLE CODE**  | "fix", "add", "change" (single file)       | TIER 0 + TIER 1 (lite)              | Inline Edit                 |
| **COMPLEX CODE** | "build", "create", "implement", "refactor" | TIER 0 + TIER 1 (full) + Agent tool | **{task-slug}.md Required** |
| **DESIGN/UI**    | "design", "UI", "page", "dashboard"        | TIER 0 + TIER 1 + Agent tool        | **{task-slug}.md Required** |

---

## INTELLIGENT AGENT ROUTING (STEP 2 - AUTO)

**ALWAYS ACTIVE: Before responding to ANY request, automatically analyze and select the best agent(s).**

### Auto-Selection Protocol

1. **Analyze (Silent)**: Detect domains (Frontend, Backend, Security, etc.) from user request.
2. **Select Agent(s)**: Choose the most appropriate specialist(s).
3. **Read Agent File**: Use Read tool on `.agent/agents/{agent-name}.md`.
4. **Load Skills**: Read SKILL.md files listed in the agent's frontmatter `skills:` field.
5. **Apply**: Generate response using the selected agent's persona and rules.

### AGENT ROUTING CHECKLIST (MANDATORY BEFORE EVERY CODE/DESIGN RESPONSE)

| Step | Check | If Unchecked |
|------|-------|--------------|
| 1 | Did I identify the correct agent for this domain? | STOP. Analyze request domain first. |
| 2 | Did I READ the agent's `.md` file? | STOP. Read `.agent/agents/{agent}.md` |
| 3 | Did I load required skills from agent's frontmatter? | STOP. Check `skills:` field and read them. |
| 4 | Am I applying the principles, not just copying patterns? | STOP. Understand WHY, then apply. |

**Failure Conditions:**
- Writing code without identifying an agent = PROTOCOL VIOLATION
- Ignoring agent-specific rules = QUALITY FAILURE

---

## TIER 0: UNIVERSAL RULES (Always Active)

### Language Handling

When user's prompt is NOT in English:
1. **Internally translate** for better comprehension
2. **Respond in user's language** - match their communication
3. **Pay extra attention to native characters** — e.g. Turkish: always use ç, ğ, ı, İ, ö, ş, ü correctly. Never fall back to ASCII equivalents.
4. **Code comments/variables** remain in English

### Clean Code (Global Mandatory)

**ALL code MUST follow `skills/clean-code` rules. No exceptions.**

- **Code**: Concise, direct, no over-engineering. Self-documenting.
- **Testing**: Mandatory. Pyramid (Unit > Int > E2E) + AAA Pattern.
- **Performance**: Measure first. Adhere to 2025 standards (Core Web Vitals).
- **Infra/Safety**: 5-Phase Deployment. Verify secrets security.

### File Dependency Awareness

**Before modifying ANY file:**
1. Check `CODEBASE.md` if it exists for file dependencies
2. Identify dependent files
3. Update ALL affected files together

### Hooks (Automated Guardrails)

**Active automatically via `.claude/settings.json`.** No manual intervention needed.

- **PreToolUse:Bash** → `dangerous_cmd_check.sh` blocks destructive commands
- **PreToolUse:Write|Edit** → `secret_scanner.sh` blocks hardcoded secrets
- **PostToolUse:Edit|Write** → `lint_check.sh` runs lint after file changes
- **Stop** → `session_save.sh` persists session context

### Always-On Rules

**MANDATORY:** Rules in `.agent/rules/common/` apply to ALL code output:
- `git-workflow.md` - Conventional Commits, branch naming
- `coding-style.md` - Naming conventions, file organization
- `testing.md` - AAA pattern, coverage targets
- `security.md` - Secrets, OWASP, input validation
- `performance.md` - Core Web Vitals, bundle limits

### Contexts (Mode-Specific)

Load appropriate context from `.agent/contexts/` based on task:
- `dev.md` - Development/prototyping mode
- `review.md` - Code review mode
- `research.md` - Analysis/comparison mode

### System Map Read

**MANDATORY:** Read `.agent/ARCHITECTURE.md` at session start to understand Agents, Skills, Scripts, Hooks, Rules, and Contexts.

**Path Awareness:**
- Agents: `.agent/agents/`
- Skills: `.agent/skills/`
- Runtime Scripts: `.agent/skills/<skill>/scripts/`
- Hooks: `.agent/scripts/hooks/`
- Rules: `.agent/rules/common/`
- Contexts: `.agent/contexts/`
- PM Detection: `.agent/scripts/detect_pm.py`

### Read -> Understand -> Apply

```
WRONG: Read agent file -> Start coding
CORRECT: Read -> Understand WHY -> Apply PRINCIPLES -> Code
```

**Before coding, answer:**
1. What is the GOAL of this agent/skill?
2. What PRINCIPLES must I apply?
3. How does this DIFFER from generic output?

---

## TIER 1: CODE RULES (When Writing Code)

### Project Type Routing

| Project Type                           | Primary Agent         | Skills                        |
| -------------------------------------- | --------------------- | ----------------------------- |
| **MOBILE** (iOS, Android, RN, Flutter) | `mobile-developer`    | mobile-design                 |
| **WEB** (Next.js, React web)           | `frontend-specialist` | frontend-design               |
| **BACKEND** (API, server, DB)          | `backend-specialist`  | api-patterns, database-design |
| **GAME** (Unity, Godot, Phaser)        | `game-developer`      | game-development              |

> Mobile + frontend-specialist = WRONG. Mobile = mobile-developer ONLY.

### Socratic Gate

**For complex requests, STOP and ASK first:**

| Request Type            | Strategy       | Required Action                                                   |
| ----------------------- | -------------- | ----------------------------------------------------------------- |
| **New Feature / Build** | Deep Discovery | ASK minimum 3 strategic questions                                 |
| **Code Edit / Bug Fix** | Context Check  | Confirm understanding + ask impact questions                      |
| **Vague / Simple**      | Clarification  | Ask Purpose, Users, and Scope                                     |
| **Full Orchestration**  | Gatekeeper     | STOP subagents until user confirms plan details                   |

**Protocol:**
1. **Never Assume:** If even 1% is unclear, ASK.
2. **Wait:** Do NOT invoke subagents or write code until the user clears the Gate.
3. **Reference:** Full protocol in `skills/brainstorming`.

### Final Checklist Protocol

**Trigger:** When the user says "son kontrolleri yap", "final checks", or similar phrases.

| Task Stage       | Command                                            | Purpose                        |
| ---------------- | -------------------------------------------------- | ------------------------------ |
| **Manual Audit** | `python .agent/scripts/checklist.py .`             | Priority-based project audit   |
| **Pre-Deploy**   | `python .agent/scripts/verify_all.py . --url <URL>`| Full Suite + Performance + E2E |

**Priority Execution Order:**
1. **Security** -> 2. **Lint** -> 3. **Schema** -> 4. **Tests** -> 5. **UX** -> 6. **SEO** -> 7. **Lighthouse/E2E**

**Rules:**
- **Completion:** A task is NOT finished until `checklist.py` returns success.
- **Reporting:** If it fails, fix the **Critical** blockers first (Security/Lint).

---

## CLAUDE CODE SPECIFIC: AGENT INVOCATION

### Using the Agent Tool for Specialists

When a task requires a specialist agent, use Claude Code's native Agent tool:

```
Agent tool -> subagent_type: "general-purpose"
Prompt: "You are acting as the {agent-name} from Bilge Development Kit.
Read .agent/agents/{agent-name}.md first.
Load skills listed in the frontmatter.
Then execute: {specific task description}
Apply all rules from the agent file and its skills."
```

### Multi-Agent Orchestration Protocol

For complex tasks (3+ domains):
1. Read `.agent/agents/orchestrator.md`
2. Phase 1 (Planning): Use project-planner agent to create `{task-slug}.md`
3. Checkpoint: Get user approval on the plan
4. Phase 2 (Implementation): Invoke specialist agents in parallel via Agent tool
5. Phase 3 (Verification): Run validation scripts

### Context Passing (MANDATORY for subagents)

When invoking ANY subagent, include:
1. **Original User Request:** Full text
2. **Decisions Made:** All user answers to questions
3. **Previous Agent Work:** Summary of prior work
4. **Current Plan State:** Reference to plan files if they exist

---

## TIER 2: DESIGN RULES (Reference)

Design rules are in the specialist agents, NOT here.

| Task         | Read Agent File                  |
| ------------ | -------------------------------- |
| Web UI/UX    | `.agent/agents/frontend-specialist.md` |
| Mobile UI/UX | `.agent/agents/mobile-developer.md`    |

**These agents contain:**
- Purple Ban (no violet/purple colors)
- Template Ban (no standard layouts)
- Anti-cliche rules
- Deep Design Thinking protocol

> For design work: Open and READ the agent file. Rules are there.

---

## QUICK REFERENCE

### Agents & Skills
- **Masters**: `orchestrator`, `project-planner`, `security-auditor`, `backend-specialist`, `frontend-specialist`, `mobile-developer`, `debugger`, `game-developer`
- **Key Skills**: `clean-code`, `brainstorming`, `app-builder`, `frontend-design`, `mobile-design`, `plan-writing`

### Key Scripts
- **Verify**: `.agent/scripts/verify_all.py`, `.agent/scripts/checklist.py`
- **Detection**: `.agent/scripts/detect_pm.py`
- **Preview**: `.agent/scripts/auto_preview.py`
- **Session**: `.agent/scripts/session_manager.py`

---
> Source: [bugrabilge/bilge-development-kit](https://github.com/bugrabilge/bilge-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
