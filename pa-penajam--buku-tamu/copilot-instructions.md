## buku-tamu

> This file contains mandatory instructions that Claude Code MUST follow in every working session on this project. All rules are **non-negotiable** and must not be ignored under any circumstances.

# CLAUDE.md

## Agent Core Rules

This file contains mandatory instructions that Claude Code MUST follow in every working session on this project. All rules are **non-negotiable** and must not be ignored under any circumstances.

---

## 1. Response Language: Bahasa Indonesia (MANDATORY)

### Strict Rules

- **ALWAYS** respond in Bahasa Indonesia for ALL communications — no exceptions.
- This applies to: explanations, clarification questions, error messages, suggestions, technical discussions, commit message descriptions, debugging explanations, and every other form of communication with the user.
- **NEVER** switch to English or any other language unless the user explicitly requests it.
- If unsure, default to Bahasa Indonesia.

### Exceptions (Remain in English)

The following elements should remain in English as they are part of programming conventions:

- Variable names, function names, class names, and code identifiers.
- Programming syntax and language-specific keywords.
- Library, package, framework, and tool names.
- Git commit messages (optional — follow user preference).
- Technical configuration file contents (package.json, tsconfig.json, etc.).

### Example

```
✅ Correct:
"Saya akan membuat komponen React untuk halaman login. Pertama, mari kita install dependency yang dibutuhkan..."

❌ Wrong:
"I'll create a React component for the login page. First, let's install the required dependencies..."
```

### Code Comments

- All inline code comments MUST be written in **Bahasa Indonesia**.
- JSDoc/TSDoc: descriptions in Bahasa Indonesia, type annotations remain in English.

```typescript
/**
 * Mengambil data pengguna berdasarkan ID.
 * Mengembalikan null jika pengguna tidak ditemukan.
 *
 * @param userId - ID unik pengguna
 * @returns Data pengguna atau null
 */
async function fetchUserData(userId: string): Promise<User | null> {
  // Validasi input sebelum melakukan query ke database
  if (!userId) return null;

  // Ambil data dari database
  const user = await db.users.findUnique({ where: { id: userId } });
  return user;
}
```

### Communication Tone

- Explain every technical decision in Bahasa Indonesia.
- If a breaking change or deprecation is found, inform the user in Bahasa Indonesia with alternative solutions.
- When suggesting architectural changes, provide clear reasoning in Bahasa Indonesia.
- Error messages directed at the user must be in Bahasa Indonesia.
- Internal technical logs may remain in English.

---

## 2. Mandatory Use of Context7 MCP

### Strict Rules

- **ALWAYS** use Context7 MCP to fetch current documentation before writing any code that involves external libraries, frameworks, or tools.
- This applies to **all** of the following situations:
  - Using libraries/frameworks already present in the project.
  - Adding new libraries/frameworks to the project.
  - Upgrading or migrating library versions.
  - Writing code that depends on any external library API.
  - Fixing bugs related to library usage.
  - Configuring build tools, bundlers, or dev tooling.

### How to Use

Append `use context7` to every internal prompt when documentation reference is needed. Context7 will automatically fetch up-to-date, version-specific documentation from official sources.

### Required Workflow

```
1. User requests a feature/change involving the tech stack
2. MANDATORY: Use Context7 to fetch current documentation
3. Verify that APIs, methods, and patterns match the current version
4. Only then write/modify code based on accurate documentation
5. If there is a conflict between internal knowledge and Context7, ALWAYS PRIORITIZE Context7 results
```

### Situations Requiring Context7

| Situation | Context7 Action |
|---|---|
| Creating a new React component | Check React docs for the project's installed version |
| Setting up Next.js routing | Check App Router vs Pages Router for the installed Next.js version |
| Configuring Tailwind CSS | Check config syntax for the current Tailwind version |
| Querying database with Prisma | Check Prisma Client API for the installed version |
| Adding Express middleware | Check current Express middleware patterns |
| Setting up authentication | Check docs for the auth library in use (NextAuth, Clerk, etc.) |
| Third-party API integration | Check the latest SDK/library wrapper version |
| Configuring Vite/Webpack/Turbopack | Check bundler docs for current configuration API |

### Prohibited Actions

```
❌ Writing code based on training data without Context7 verification
❌ Assuming an API or method is still valid without checking current docs
❌ Skipping Context7 because you believe you already "know" a library
❌ Using patterns or syntax that may have been deprecated
```

---

## 3. Context7 MCP Configuration

### Setup via CLI (Recommended)

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

### Setup via Remote HTTP

```bash
claude mcp add --transport http context7 https://mcp.context7.com/mcp
```

### Manual Setup (claude_desktop_config.json or .mcp.json)

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Setup with API Key (Higher Rate Limits)

Get a free API key at [context7.com/dashboard](https://context7.com/dashboard).

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest", "--api-key", "YOUR_API_KEY"]
    }
  }
}
```

### Verify Connection

```bash
claude mcp list
```

Ensure the output shows `context7` with `✓ Connected` status.

---

## 4. Mandatory Coding Principles

The following coding principles **MUST** be applied in every piece of code that is written, modified, or reviewed. The goal is to produce software that is maintainable, scalable, readable, and minimizes technical debt for long-term collaboration.

### SOLID Principles (Object-Oriented Design)

- **Single Responsibility (SRP)**: A class or module should have one, and only one, reason to change. If a class handles more than one responsibility, split it into separate classes.

```typescript
// ❌ Wrong: One class handling multiple responsibilities
class UserService {
  createUser() { /* ... */ }
  sendEmail() { /* ... */ }
  generateReport() { /* ... */ }
}

// ✅ Correct: Each class has a single responsibility
class UserService {
  createUser() { /* ... */ }
}
class EmailService {
  sendEmail() { /* ... */ }
}
class ReportService {
  generateReport() { /* ... */ }
}
```

- **Open/Closed (OCP)**: Software entities should be open for extension but closed for modification. Use abstractions so new features can be added without altering existing code.

- **Liskov Substitution (LSP)**: Subtypes must be substitutable for their base types without altering program correctness. Every derived class must fulfill the contract of its parent class.

- **Interface Segregation (ISP)**: Clients should not be forced to depend on methods they do not use. Split large interfaces into smaller, focused ones.

```typescript
// ❌ Wrong: Interface is too broad
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// ✅ Correct: Small and focused interfaces
interface Workable {
  work(): void;
}
interface Feedable {
  eat(): void;
}
```

- **Dependency Inversion (DIP)**: Depend on abstractions (interfaces), not concrete implementations. High-level modules must not depend directly on low-level modules.

```typescript
// ❌ Wrong: Direct dependency on concrete implementation
class OrderService {
  private mysqlDb = new MySQLDatabase();
}

// ✅ Correct: Depends on an abstraction
class OrderService {
  constructor(private db: DatabaseInterface) {}
}
```

### Clean Code & Design Principles

- **DRY (Don't Repeat Yourself)**: Avoid duplication of logic. If the same code exists in two or more places, extract it into a reusable function or module.

- **KISS (Keep It Simple, Stupid)**: Avoid unnecessary complexity. Prioritize readability and simplicity. A simple working solution beats a complex "elegant" one.

- **YAGNI (You Ain't Gonna Need It)**: Do not add functionality until it is actually needed. Do not write code for hypothetical future requirements.

- **Clean Code / Readability**: Use meaningful variable and function names, consistent formatting, and simple logic. Code should be self-documenting.

- **Abstraction**: Hide internal complexity and only expose necessary interfaces. Abstraction layers help isolate changes and reduce coupling.

### Best Practices for Implementation

- **Test-Driven Development (TDD)**: Write tests before implementing code. Follow RED-GREEN-REFACTOR strictly.

- **Continuous Refactoring**: Regularly improve code structure without changing behavior. Boy Scout Rule: leave code better than you found it.

- **Code Reviews**: Utilize peer reviews to maintain code quality, standards, and consistency.

- **Security by Design**: Validate all input, sanitize data, and manage dependencies regularly. Security is a foundation, not an afterthought.

```
Basic security checklist:
- Validate and sanitize all user input
- Use parameterized queries (prevent SQL injection)
- Never hardcode secrets or credentials in source code
- Update dependencies regularly for security patches
- Apply the principle of least privilege
```

### Agent Enforcement Table

| Principle | How the Agent Applies It |
|---|---|
| SRP | Every function/class/module handles only one responsibility |
| OCP | Use patterns that allow extension without modification |
| LSP | Ensure inheritance and polymorphism are correctly applied |
| ISP | Create small, focused interfaces |
| DIP | Inject dependencies through constructors or parameters |
| DRY | Identify and eliminate code duplication |
| KISS | Choose the simplest solution that fulfills the requirement |
| YAGNI | Only implement what is requested — nothing more |
| TDD | Write tests before implementation — always |
| Security | Always validate input and follow security best practices |

---

## 5. Mandatory Superpowers Workflow (ALL Skills)

The agent MUST use the **full Superpowers skills framework** throughout the entire development lifecycle. Superpowers is not just brainstorming — it is a **complete software development methodology** with skills for every phase. The agent must check for and activate relevant skills before any task.

### Core Rules

- **NEVER** enter the built-in plan mode. The Superpowers workflow **fully replaces** it.
- **ALWAYS** check which Superpowers skill is relevant before starting any task.
- Skills are **mandatory workflows, not suggestions**. When a skill exists for a task, using it is required.
- **The agent checks for relevant skills before any task.**

---

### The Complete Development Workflow

```
1. BRAINSTORMING              — Refine ideas, explore requirements
       ↓
2. USING-GIT-WORKTREES        — Create isolated branch & workspace
       ↓
3. WRITING-PLANS              — Break work into micro-tasks with TDD
       ↓
4. EXECUTING-PLANS             — Batch execution with human checkpoints
   or SUBAGENT-DRIVEN-DEV      — Parallel subagents with two-stage review
   (+ DISPATCHING-PARALLEL-AGENTS for concurrent independent tasks)
       ↓
5. TEST-DRIVEN-DEVELOPMENT    — RED → GREEN → REFACTOR (every task)
       ↓
6. REQUESTING-CODE-REVIEW     — Review against plan, block on critical issues
   + RECEIVING-CODE-REVIEW     — Respond to feedback, apply fixes
       ↓
7. FINISHING-A-DEV-BRANCH     — Verify tests, merge/PR/keep/discard, cleanup

DEBUGGING (anytime):
   SYSTEMATIC-DEBUGGING        — 4-phase root cause process
   + VERIFICATION-BEFORE-COMPLETION — Prove it's actually fixed
```

**Never skip any phase. Always start from brainstorming.**

---

### Collaboration Skills

#### `brainstorming`
**Trigger:** Activates before any code is written. Intercepts plan mode.

- Engage in a Socratic dialogue with the user to deeply understand requirements.
- Ask targeted clarification questions (1-2 at a time).
- Identify edge cases, constraints, and potential pitfalls.
- Explore existing codebase and project context before proposing solutions.
- Present 2-3 design approaches with clear trade-offs.
- Present the design in short, readable sections (200-300 words each).
- Save the design document.
- **Wait for user approval** before proceeding.
- All communication in **Bahasa Indonesia**.

```
Example brainstorming questions:
- "Apa tujuan utama fitur ini? Siapa yang akan menggunakannya?"
- "Apakah ada batasan performa atau kompatibilitas yang perlu diperhatikan?"
- "Berikut 3 pendekatan yang bisa kita gunakan. Mana yang paling sesuai?"
```

#### `writing-plans`
**Trigger:** Activates with an approved design document.

- Break work into micro-tasks (2-5 minutes each).
- Every task: exact file paths, complete code expectations, verification steps.
- Clear acceptance criteria per task.
- Define implementation order and dependencies.
- Include test requirements (TDD approach).
- Emphasize YAGNI and DRY.
- Present plan to user for approval before execution.

#### `executing-plans`
**Trigger:** Activates with an approved plan.

- Execute tasks in defined batches with human checkpoints.
- Follow TDD for each task.
- Pause at each checkpoint for user review.

#### `subagent-driven-development`
**Trigger:** Alternative to executing-plans for faster parallel work.

- Dispatch a fresh subagent per task.
- Two-stage review: spec compliance check → code quality check.
- Can work autonomously for extended periods without deviating from the plan.

#### `dispatching-parallel-agents`
**Trigger:** When independent tasks can run concurrently.

- Run concurrent subagent workflows.
- Coordinate results when tasks complete.

#### `requesting-code-review`
**Trigger:** Activates between tasks during execution.

- Run through a pre-review checklist before submitting.
- Review implementation against the plan.
- Report issues by severity (info, warning, critical).
- **Critical issues block progress** until resolved.

#### `receiving-code-review`
**Trigger:** When review feedback is received.

- Respond to feedback constructively.
- Apply fixes and re-verify before moving on.

---

### Development Skills

#### `using-git-worktrees`
**Trigger:** Activates after design approval.

- Create an isolated workspace on a new branch.
- Run project setup in the worktree.
- Verify a clean test baseline before implementation.
- Enables parallel development without branch conflicts.

#### `finishing-a-development-branch`
**Trigger:** Activates when all tasks are complete.

- Verify all tests pass.
- Present options: merge, create PR, keep working, or discard.
- Clean up the worktree after the decision.

---

### Testing Skill

#### `test-driven-development`
**Trigger:** Activates during ALL implementation — not optional.

- Enforce RED-GREEN-REFACTOR strictly:
  1. **RED**: Write a failing test first.
  2. **GREEN**: Write the minimum code to make the test pass.
  3. **REFACTOR**: Clean up without changing behavior, then commit.
- **If code is written before tests, delete the code and start over.** No exceptions.
- Reference testing anti-patterns to avoid common mistakes.

---

### Debugging Skills

#### `systematic-debugging`
**Trigger:** When bugs are encountered.

- 4-phase root cause process:
  1. **Observe**: Gather evidence, reproduce the bug.
  2. **Hypothesize**: Form theories about the root cause.
  3. **Test**: Validate hypotheses with targeted experiments.
  4. **Fix**: Apply the fix based on confirmed root cause.
- Includes root-cause-tracing, defense-in-depth, and condition-based-waiting techniques.
- **NEVER** make random changes hoping they will work.

#### `verification-before-completion`
**Trigger:** After any fix is applied.

- Verify the fix actually resolves the issue.
- Never declare a bug fixed without evidence (passing tests, scenario resolved).
- Run full relevant test suite to check for regressions.

---

### Meta Skills

#### `writing-skills`
**Trigger:** When creating new custom skills for the project.

- Follow Superpowers skill-writing best practices.
- Includes testing methodology for validating skills.

#### `using-superpowers`
**Trigger:** Loaded at session start.

- Introduction to the entire Superpowers system.
- Teaches the agent how all skills interconnect and when to activate each one.

---

### Complete Skills Reference Table

| Skill | Category | When It Activates |
|---|---|---|
| `brainstorming` | Collaboration | Before any code; intercepts plan mode |
| `writing-plans` | Collaboration | With approved design; micro-task breakdown |
| `executing-plans` | Collaboration | With approved plan; batch execution + checkpoints |
| `subagent-driven-development` | Collaboration | With approved plan; parallel subagent execution |
| `dispatching-parallel-agents` | Collaboration | Independent tasks that can run concurrently |
| `requesting-code-review` | Collaboration | Between tasks; review against plan |
| `receiving-code-review` | Collaboration | When responding to review feedback |
| `using-git-worktrees` | Development | After design approval; isolated branch |
| `finishing-a-development-branch` | Development | All tasks complete; merge/PR/cleanup |
| `test-driven-development` | Testing | During ALL implementation; RED-GREEN-REFACTOR |
| `systematic-debugging` | Debugging | When bugs encountered; 4-phase root cause |
| `verification-before-completion` | Debugging | After any fix; prove it works |
| `writing-skills` | Meta | Creating new custom skills |
| `using-superpowers` | Meta | Session start; system introduction |

---

### Plan Mode Interception Rule

```
IF agent is about to enter plan mode:
  → CHECK: Has brainstorming been completed for this task?
    → IF NO:  Activate brainstorming skill FIRST. Do NOT enter plan mode.
    → IF YES: Proceed to writing-plans skill. Do NOT use built-in plan mode.

The built-in plan mode is FULLY REPLACED by the Superpowers workflow.
Never use the generic plan mode under any circumstance.
```

### Prohibited Actions

```
❌ Entering plan mode without completing brainstorming first
❌ Skipping brainstorming because the task "seems simple"
❌ Writing code before the plan is approved by the user
❌ Using the generic built-in plan mode instead of Superpowers
❌ Writing code before writing tests (violates TDD)
❌ Making random changes to fix bugs without systematic-debugging
❌ Declaring a bug fixed without verification-before-completion
❌ Skipping code review between tasks
❌ Implementing without user approval at each phase gate
❌ Ignoring any Superpowers skill when it is relevant to the current task
```

### Superpowers Installation

```bash
# Install in Claude Code
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# Verify commands are available
/help
# Should show:
#   /superpowers:brainstorm    - Interactive design refinement
#   /superpowers:write-plan    - Create implementation plan
#   /superpowers:execute-plan  - Execute plan in batches

# Update to latest version
/plugin update superpowers
```

---

## 6. Additional Rules

### Code Quality

- Always follow best practices from official documentation (fetched via Context7).
- Use TypeScript if the project already uses TypeScript.
- Add adequate error handling in all code paths.
- Write clean, readable, and well-documented code.

### Error Handling

- Error messages shown to users should be descriptive and in Bahasa Indonesia.
- Internal technical logs may use English conventions.
- When debugging, explain the process and findings in Bahasa Indonesia.

---

## Quick Reference

| Rule | Application |
|---|---|
| Response language | **Always Bahasa Indonesia** |
| Code language | English (variables, functions, syntax) |
| Code comments | Bahasa Indonesia |
| Context7 | Mandatory for every tech stack reference |
| Documentation priority | Context7 > internal/training knowledge |
| Plan mode | **NEVER use built-in plan mode** — Superpowers replaces it |
| Development workflow | Brainstorm → Worktree → Plan → Execute → TDD → Review → Finish |
| Superpowers skills | ALL 14 skills are mandatory when relevant |
| Coding principles | SOLID, DRY, KISS, YAGNI — always enforced |
| Testing | TDD: RED-GREEN-REFACTOR — no code before tests |
| Debugging | systematic-debugging + verification-before-completion |
| Security | Security by design — validate input, sanitize data |
| Refactoring | Continuous improvement — Boy Scout Rule |
| Communication with user | **Always in Bahasa Indonesia** |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PA-Penajam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PA-Penajam)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
