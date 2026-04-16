## date-management-app

> **Node.js/Express/TypeScript Development Guide**


# AGENTS.md
**Node.js/Express/TypeScript Development Guide**

**Version:** 1.2.3  
**Status:** Canonical guide for AI-assisted Node/Express/TypeScript development  
**Last Updated:** February 2026

***

## Table of Contents

1. **Compliance Core Rules**
2. **Express/TypeScript Development Standards**
3. **Session Startup Context**
4. **Project Structure Memory Bank**
5. **OpenSpec Workflow**
6. **Task Management with OpenSpec**
7. **Quality & Testing**
8. **Memory Management**
9. **Security Code Review**
10. **Troubleshooting**

***

### Note on SKILLS/AGENTS
SKILLs and AGENTS live in .github/ for VSCode and .agents/ for Windsurf and can should be used when the work needs specific knowledge that a skill or agent contains 

### Note on MCP
**Always** check for tools and MCP servers to assist with modifications e.g. use the shadcn-UI-mcp to find default components rather than building from scratch.

***

## 1. Compliance Core Rules

### Startup Compliance Output (Every Session)

| Rule | Requirement | Validation |
|------|------------|-----------|
| ❌ **No new files without reuse analysis** | Search codebase, check existing controllers/services, provide justification | "Analyzed `src/controllers/X`, `src/services/Y`. Cannot extend because [reason]" |
| ❌ **No rewrites when refactoring possible** | Prefer incremental improvements to existing services/controllers | "Extending `User` service at line X rather than creating new service" |
| ❌ **No ignoring Express/TS conventions** | Follow Express REST patterns, layered architecture (routes → controllers → services → data access), TypeScript strict types, CRA conventions for React | "Follows Express patterns: routes in `src/routes/`, controllers in `src/controllers/`, services in `src/services/`, DB access in `src/db/` or `src/repositories/`; React follows CRA `src/` structure" |
| ❌ **No skipping tests** | TDD mandatory—red, green, refactor. Never commit without tests | "Red: wrote failing test \| Green: implemented \| Verified: exit code 0" |

### The Four Sacred Rules (Express/TypeScript-Specific)

- **Approval Gates**: No commits without explicit user approval
- **Sandbox First**: All work in feature branches, never `main`/`master`
- **Citations**: Always `src/path/file.ts:42` (single line) or `src/path/file.ts:42-58` (range)
- **No Mock Data**: Never fake/simulated data in production; test fixtures are OK
- **No Secrets**: Never hardcode API keys, passwords, or credentials
- **TDD Mandatory**: Tests written before production code
- **Code Quality**: Must pass `npm run lint`, `npm test`, and `ubs .` before commit
- **Task Tracking**: Use OpenSpec workflows for ALL work—no markdown TODOs

***

### Non-Negotiables

Following Express/TypeScript conventions reduces code. Always ask: **"Does Express/Node already provide this?"**

**Good:**
- **Route**: `src/routes/users.ts` defining endpoints
- **Controller**: `src/controllers/usersController.ts` handling requests/responses
- **Service**: `src/services/usersService.ts` containing business logic
- **Repository/DB**: `src/db/usersRepository.ts` or `src/repositories/usersRepository.ts` for data access
- **Model/Types**: `src/models/User.ts` or `src/types/User.ts` for TypeScript interfaces/types

**Anti-Pattern:**
- Custom database layer bypassing established patterns
- Non-standard directory structures
- Controllers with business logic (should be in services)
- Missing TypeScript types

***

## 2. Express/TypeScript Development Standards

### Core Express Principles

#### Layered Architecture (Separation of Concerns)

| Component | Responsibility | Anti-Pattern |
|-----------|---------------|--------------|
| **Route** | Define endpoints, map HTTP methods to controllers | Business logic in routes |
| **Controller** | Handle requests/responses, validate input, call services | Direct database queries, business logic |
| **Service** | Business logic, orchestration, data transformation | HTTP-specific code, direct DB queries |
| **Repository/Data Access** | Database operations, queries, transactions | Business logic |
| **Model/Type** | TypeScript interfaces/types, data shape validation | Logic implementation |

#### RESTful Design

Actions correspond to HTTP verbs:

```typescript
// src/routes/users.ts
import express from 'express';
import { usersController } from '../controllers/usersController';

const router = express.Router();

router.get('/', usersController.index);        // GET /users - list all
router.get('/:id', usersController.show);      // GET /users/:id - single resource
router.post('/', usersController.create);      // POST /users - create new
router.put('/:id', usersController.update);    // PUT /users/:id - full update
router.patch('/:id', usersController.update);  // PATCH /users/:id - partial update
router.delete('/:id', usersController.destroy);// DELETE /users/:id - delete

export default router;
```

#### TypeScript Strict Types

All code must use strict TypeScript:

```typescript
// src/types/User.ts
export interface User {
  id: number;
  email: string;
  name: string;
  createdAt: Date;
}

export interface CreateUserDTO {
  email: string;
  name: string;
}
```

Never use `any` without justification. Use proper types or `unknown` with type guards.

#### Thin Controllers, Logic in Services

**Good:**

```typescript
// src/controllers/usersController.ts - Coordination only
import { Request, Response } from 'express';
import { usersService } from '../services/usersService';

export const usersController = {
  async create(req: Request, res: Response) {
    try {
      const user = await usersService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
};

// src/services/usersService.ts - Business logic
import { usersRepository } from '../db/usersRepository';
import { CreateUserDTO } from '../types/User';

export const usersService = {
  async createUser(userData: CreateUserDTO) {
    // Validate business rules
    if (!userData.email.includes('@')) {
      throw new Error('Invalid email format');
    }
    
    // Create user
    const user = await usersRepository.create(userData);
    
    // Send welcome email (business logic)
    await emailService.sendWelcome(user.email);
    
    return user;
  }
};
```

**Anti-Pattern:**

```typescript
// Controller with business logic - DON'T DO THIS
export const usersController = {
  async create(req: Request, res: Response) {
    // Email validation in controller - should be in service!
    if (!req.body.email.includes('@')) {
      return res.status(400).json({ error: 'Invalid email' });
    }
    
    const user = await usersRepository.create(req.body);
    await emailService.sendWelcome(user.email); // Business logic in controller!
    res.json(user);
  }
};
```

#### Single Responsibility Principle (SRP)

Each module/class should have one reason to change.

**Good:**

```typescript
// src/services/userRegistration.ts
import { CreateUserDTO, User } from '../types/User';
import { usersRepository } from '../db/usersRepository';
import { emailService } from './emailService';

export class UserRegistration {
  constructor(
    private userRepo = usersRepository,
    private emailSvc = emailService
  ) {}

  async register(userData: CreateUserDTO): Promise<User> {
    const user = await this.userRepo.create(userData);
    await this.emailSvc.sendConfirmation(user.email);
    return user;
  }
}
```

**Anti-Pattern:**

```typescript
// Service doing too much
export class UserService {
  async registerAndEmailAndNotifyAdminAndLog(data: any) {
    // Too many responsibilities!
  }
}
```

#### Dependency Injection

Pass dependencies as parameters, don't hardcode them:

```typescript
// Good - Dependency injected
export class UserService {
  constructor(private emailService: EmailService) {}

  async createAndNotify(params: CreateUserDTO) {
    const user = await usersRepository.create(params);
    await this.emailService.notify(user);
    return user;
  }
}

// Testing it
describe('UserService', () => {
  it('sends email on create', async () => {
    const mockEmailService = { notify: jest.fn() };
    const service = new UserService(mockEmailService);
    
    await service.createAndNotify({ email: 'test@example.com', name: 'Test' });
    
    expect(mockEmailService.notify).toHaveBeenCalled();
  });
});
```

### Clean Code Rules

- **Intention-Revealing Names**: `activeUsersCount` not `x` or `getUsers`
- **Single Responsibility**: Each function one clear purpose
- **Guard Clauses First**: Return early for edge cases
- **Constants**: `STATUS.ACTIVE` not `'active'` string literals
- **Input → Process → Return**: Clear structure
- **Fail with Specific Errors**: Throw custom errors, not generic ones
- **Comments Explain Why**: Why, not what (code shows what)

**Good:**

```typescript
async function activateUser(user: User): Promise<User> {
  // Guard clause - early return
  if (user.isActive) {
    return user;
  }
  
  try {
    user.activatedAt = new Date();
    await usersRepository.update(user);
    await emailService.sendWelcome(user);
    return user;
  } catch (error) {
    logger.error(`Activation failed: ${error.message}`);
    throw new ActivationError(`Could not activate user ${user.id}`);
  }
}
```

### Project Structure

Use `codemap` for a quick check of the project structure:

```bash
codemap .               # Project structure
codemap --deps          # How files connect
codemap --diff          # What changed vs main
codemap --diff --ref branch  # Changes vs specific branch
```
# Agentlens Integration

This project uses **agentlens** for AI-optimized documentation.

## Reading Protocol

Follow this order to understand the codebase efficiently:

1. **Start here**: `.agentlens/INDEX.md` - Project overview and module routing
2. **AI instructions**: `.agentlens/AGENT.md` - How to use the documentation
3. **Module details**: `.agentlens/modules/{module}/MODULE.md` - File lists and entry points
4. **Before editing**: Check `.agentlens/modules/{module}/memory.md` for warnings/TODOs

## Documentation Structure

```
.agentlens/
├── INDEX.md              # Start here - global routing table
├── AGENT.md              # AI agent instructions
├── modules/
│   └── {module-slug}/
│       ├── MODULE.md     # Module summary
│       ├── outline.md    # Symbol maps for large files
│       ├── memory.md     # Warnings, TODOs, business rules
│       └── imports.md    # Dependencies
└── files/                # Deep docs for complex files
```

## During Development

- Use `.agentlens/modules/{module}/outline.md` to find symbols in large files
- Check `.agentlens/modules/{module}/imports.md` for dependencies
- For complex files, see `.agentlens/files/{file-slug}.md`

## Commands

| Task | Command |
|------|---------|
| Regenerate docs | `agentlens` |
| Fast update (changed only) | `agentlens --diff main` |
| Check if stale | `agentlens --check` |
| Force full regen | `agentlens --force` |

## Key Patterns

- **Module boundaries**: `mod.rs` (Rust), `index.ts` (TS), `__init__.py` (Python)
- **Large files**: >500 lines, have symbol outlines
- **Complex files**: >30 symbols, have L2 deep docs
- **Hub files**: Imported by 3+ files, marked with 🔗
- **Memory markers**: TODO, FIXME, WARNING, SAFETY, RULE

## 3. Session Startup Context

### Load Priority Based on Task Complexity

**Every Session (Mandatory):**

1. Output compliance statement (Section 1)
2. Load AGENTS.md
3. Load relevant documentation (see below)
4. Run `git standup -d 7` and check the project's commits over the past seven (7) days
5. Identify environment (development/test/staging/production)
6. Check OpenSpec changes: `openspec list` (see active work)

**Quick Bug Fix (< 30 min):**
- Load relevant controller/service/repository
- Understand failing test (if applicable)
- Load affected routes
- Check OpenSpec: `openspec list` (active changes)

**Standard Feature Work (2-4 hours):**
- Load `README.md` (project overview)
- Load database schema from `src/db/schema.sql` or migration files
- Load relevant controllers, services, repositories
- Load existing tests (same area) as reference
- Check OpenSpec: `openspec list` and `openspec list --specs` (get context)

**Architecture/Refactoring (4+ hours):**
- Load all of above
- Load architecture documentation
- Load decision logs (if exists)
- Review existing patterns in codebase
- Check OpenSpec: `openspec list` and review `openspec/project.md`

**Package-Specific Work:**
- Load `packages.md` when working with Express middleware, authentication, testing libraries, etc.

### Session Question Protocol

Before starting work, clarify:

1. **Task**: What is the clear objective?
2. **Scope**: Which controllers/services/repositories affected?
3. **Success**: What does "done" look like? Acceptance criteria?
4. **Constraints**: Any architectural requirements or limitations?
5. **Context**: What related work exists? PRs, issues, patterns

***

## 4. Project Structure 
### Documentation Standards

**README.md** should include:
- What this project does
- Local setup (Node version, npm/yarn, DB setup)
- How to run tests (`npm test`)
- How to run server (`npm run dev`)
- Key architecture decisions
- Contributing guidelines

**docs/architecture.md** should include:
- System components (routes, controllers, services, background jobs)
- Key data flows (e.g., user signup flow)
- External integrations
- Performance considerations

----------

## 5. State Machine

`PLAN → BUILD → DIFF → QA → APPROVAL → APPLY → DOCS → END  ↓       ↓                   ↓ (fail/changes/major changes needed)`

### States: PLAN → BUILD → DIFF → QA → APPROVAL → APPLY → DOCS

----------

### PLAN State (Proposal)

**In:** Task contract / Feature request **Out:** Validated OpenSpec Proposal **Exit:** User approves proposal files
**Actions:**

1.  **Search Memory (REQUIRED):** Run `node scripts/mem-recall.js "<task keywords>"` to find similar patterns, past solutions, or related work
2. Choose a concise `change-id` (e.g., `add-user-validation`).
3.  Run: `openspec proposal <change-id>`.
4.  Edit `openspec/changes/<change-id>/proposal.md` with analysis, reuse strategy, and implementation steps.
5.  Edit `openspec/changes/<change-id>/tasks.md` with the checklist of work.
6.  (Optional) Create `spec.md` deltas if requirements are changing.
7.  Validate: `openspec validate <change-id> --strict`.

**Required Content (in `proposal.md`):**

```markdown
# Proposal: [Feature/Fix Name]   
## Analysis -  
**Current**: `src/controllers/usersController.ts` 
- User controller with X endpoints 
**Affected**: `src/services/usersService.ts`, `src/__tests__/users.test.ts` 
**Pattern**: Extends existing validation middleware   

## Reuse Strategy 
- Extend User service with new validation method 
- Follow existing test pattern from `src/__tests__/posts.test.ts`   

## Implementation Steps
1. Add validation method to `usersService.ts` 
2. Update controller to use new validation 
3. Add helper to `usersRepository.ts` 
```

**Exit:** User approves the proposal files (`proposal.md` & `tasks.md`).

----------

### BUILD State (Implementation)

**In:** Approved Proposal **Out:** Code changes

**Actions:**
1. **Codemap:** Use codemap commands for context of project structure:
```bash
codemap .               # Project structure
codemap --deps          # How files connect
```
2.  **Read Tasks:** Review `openspec/changes/<change-id>/tasks.md`.
3.  **Branch (REQUIRED):** `git checkout -b feature/<change-id>`
4.  **Loop through Tasks:**
    -   Mark task "In Progress" in `tasks.md` (mentally or via status if applicable).
    -   **RED Phase:** Write failing tests.
    -   **GREEN Phase:** Implement code.
    -   **Refactor:** Clean up.
    -   Mark task `[x]` in `tasks.md`.
5.  **Scans:** Run tests and UBS.

#### TDD Phases

```typescript 
// PHASE 1: RED - Failing test
// src/__tests__/users.test.ts
describe('User', () => {
  describe('validations', () => {
    it('validates email format', () => {
      const result = usersService.validateEmail('invalid');
      expect(result.isValid).toBe(false);
    });
  });
});

// PHASE 2: GREEN - Implementation
// src/services/usersService.ts
export const usersService = {
  validateEmail(email: string): ValidationResult {
    // ... implementation
    return {
      isValid: true,
      errors: []
    };
  }
};

// Verify: npm test (should now pass)

```

**Exit:** Tests pass (`npm run test:frontend:diff` or `npm run test:backend:diff`), linter clean, `tasks.md` fully checked `[x]`.

----------

### DIFF State

**In:** BUILD complete **Out:** Diff, Test Coverage % for Diff, No Typescript errors

**Present:**

```markdown
## Proposed Changes   
**Change ID:**  `add-user-validation` 
**Tasks:** All items in `tasks.md` complete.   
**Files:** 
-  `src/services/usersService.ts`: +5 
-  `src/__tests__/users.test.ts`: +20 
-  `openspec/changes/add-user-validation/tasks.md`: Updated   

### Diff [git diff output]   
### Checks 

```bash
npm run test:backend:diff
```

- ✅ Tests: 145 passing 
- ✅ =============================== Coverage summary ===============================
Statements   : 57.71% ( 2054/3559 )
Branches     : 69.61% ( 252/362 )
Functions    : 58.25% ( 60/103 )
Lines        : 57.71% ( 2054/3559 )
================================================================================


----------

### QA State

**In:** DIFF presented **Out:** Test results, UBS reults, Linter **Exit:** Tests pass OR user waiver

**Execute:**
#### UBS (Ultimate Bug Scanner)

Flags likely bugs early. Use before every commit.

```bash
ubs <changed-files> # Specific files (1s) [RECOMMENDED] 
ubs $(git diff --name-only) # Staged files 
ubs --only=ts,js,tsx src/ # Language filter
```
**PASS**: No critical results found. 
**FAIL**: Critical warnings: [list out findings and plan of action]

- ✅ Linter: clean 
----------

### APPROVAL State (HUMAN GATE)

**In:** QA passed **Out:** User decision

**Present:**

```markdown
## Ready for Approval   
### Summary 
3. Code Review Checklist

Before committing, verify:

- ✅ Tests written first (TDD)
- ✅ >80% coverage for new code
- ✅ No any types (except justified in comments)
- ✅ Services have <50 lines per method
- ✅ Custom errors used for business logic errors
- ✅ DI used for dependencies
- ✅ No hardcoded config (use environment)
- ✅ Linter: clean 
- ✅ UBS: passed without 'Critical'
- ✅ OpenSpec: completed task marked in `tasks.md`   
**Please review. Reply with:** - "approved" / "looks good" → push to git - "change X" → Back to BUILD - "revert" → Discard all
```

----------

### APPLY State

**In:** User approved **Out:** Changes applied/merged

**Actions:**

1. Verify all tests pass (one final time). 
2. Commit Changes: Use conventional commit format:
```bash
git add .
git commit -m "feat(<area>): brief description

- Task 1 completed
- Task 2 completed

Refs: <change-id>"
```
2. Push feature branch: `git push feature/<change-id>` 
3. User creates a pull request via GitHub 
4. User raises any findings from CodeSense check
5. After PR approval and merge, confirm success

----------

### DOCS State (Archive)

**In:** APPLY succeeded **Out:** Change archived & Documentation updated

**Actions:**

1.  Confirm OpenSpec up to do `list`:
2. Store Memory: Use `node scripts/mem-log.js` to log what was accomplished:

    ```bash
    node scripts/mem-log.js FEATURE "Feature Title" "What was changed and why"
    # Or for fixes:
    node scripts/mem-log.js FIX "Fix Title" "Problem and solution"
    # Or for architectural decisions:
    node scripts/mem-log.js ARCHITECTURE "Decision Title" "Choice and rationale"
    ```
3. Update `README.md` or `docs/` if architecture changed.
4. Validate state: `openspec validate --strict`.

----------

## 6. Task Management with OpenSpec

### Essential OpenSpec Commands

Use `--json` flag for programmatic output.

```bash
# Check what to work on  
openspec list                           # Active changes with task progress  
openspec list --specs                   # List current specifications  
openspec show <change-id> --json        # View change details  
  
# Create changes (proposal stage)  
openspec proposal <change-id>           # Create new change proposal  
openspec validate <change-id> --strict  # Validate proposal formatting  
  
# Manage workflow  
openspec show <change-id>               # Review proposal, tasks, deltas  
openspec archive <change-id> --yes      # Archive completed change  
openspec validate --all                 # Validate all changes and specs
```

### Workflow: From Proposal to Archive

1.  **Start session:** `openspec list` (see active changes and progress).
2.  **Create proposal:** `openspec proposal add-feature` (scaffolds change structure).
3.  **Plan:** Edit `proposal.md` and `tasks.md` in `openspec/changes/<id>/`.
4.  **Validate:** `openspec validate <id> --strict` (ensure formatting is correct).
5.  **Implement:** Work through tasks in `tasks.md` sequentially (BUILD state).
6.  **Archive:** `openspec archive <id> --yes` (move to archive, update specs).

### Change Progress Tracking

OpenSpec tracks progress through `tasks.md` files using markdown checkboxes:

| Status | Display | Meaning |
|--------|---------|---------|
| No tasks.md | "No tasks" | Still in proposal stage |
| All `[ ]` | "3/5 tasks" | Active implementation |
| All `[x]` | "✓ Complete" | Ready for archive |

### Important Rules

-   Use **OpenSpec** for ALL change tracking (no markdown TODOs).
-   Always validate proposals: `openspec validate <id> --strict`.
-   Run `openspec list` to see active work before starting.
-   Complete tasks sequentially in `tasks.md`.
-   Archive changes after deployment: `openspec archive <id> --yes`.
-   Store change files in `openspec/changes/[change-id]/` structure.
-   Do **NOT** create planning documents in repo root.
-   Do **NOT** skip validation steps.
-   Do **NOT** archive without completing all tasks.

### Change Types

-   **Features:** New capabilities (use `## ADDED Requirements` in specs).
-   **Modifications:** Changes to existing behavior (use `## MODIFIED Requirements` in specs).
-   **Removals:** Deprecated features (use `## REMOVED Requirements` in specs).
-   **Renaming:** Requirement name changes only (use `## RENAMED Requirements` in specs).

----------

## 7. Quality & Testing

### Test-Driven Development (TDD) - Mandatory

#### Three-Phase Cycle

1.  **Red**: Write test that fails

```
// src/__tests__/users.test.ts  
it('sends welcome email when user created', async () => {  
  const user = await usersService.create({ email: 'test@example.com', name: 'Test' });  
  expect(emailService.sendWelcome).toHaveBeenCalledWith(user.email);  
});
```

2.  **Green**: Implement minimal code to pass

```
// src/services/usersService.ts  
export const usersService = {  
  async create(data: CreateUserDTO): Promise<User> {  
    const user = await usersRepository.create(data);  
    await emailService.sendWelcome(user.email);  
    return user;  
  }  
};
```

3.  **Refactor**: Improve without changing behavior

```
// src/services/usersService.ts  
export const usersService = {  
  async create(data: CreateUserDTO): Promise<User> {  
    const user = await usersRepository.create(data);  
    await this.queueWelcomeEmail(user);  
    return user;  
  },  
    
  private async queueWelcomeEmail(user: User) {  
    await emailQueue.add({ userId: user.id });  
  }  
};
```

### Completion Checklist

Before marking task complete, run all (must pass):

```
# Run all tests  
npm run test:coverage                     # Expected: high level of quality coverage  
  
# Run linter with auto-fix  
npm run lint                              # Expected: exit code 0 or only minor 
  
# Scan for bugs [CRITICAL]  
ubs $(git diff --name-only)               # Expected: exit code 0 (no critical/important issues)  
  
# Check TypeScript compilation  
npm run build                             # Expected: exit code 0  
# OR  
tsc --noEmit  
  
# Validate OpenSpec changes  
openspec validate --all                   # Expected: exit code 0
```

----------

## 8. Memory Management

## Memvid Memory System

This project uses **Memvid** as a lightweight, file-based memory layer. All project knowledge is stored in a single `project-memory.mv2` file at the project root.

### Why Memvid
- **Zero daemon overhead** — No database server, just a single file
- **Offline-first** — Works entirely locally, perfect for low-resource devices
- **Fast retrieval** — Rust-based lexical search (~1-2ms)
- **Portable** — Memory travels with the project

### Core Commands

```bash
# Store a memory
node scripts/mem-log.js <KIND> <TITLE> <MESSAGE>

# Recall memories  
node scripts/mem-recall.js <QUERY>

# Direct CLI access
memvid find project-memory.mv2 --query "search term"
memvid stats project-memory.mv2
memvid timeline project-memory.mv2
```

### Memory Kinds

| Kind | Use For |
|------|---------|
| `FIX` | Bug fixes and their solutions |
| `PATTERN` | Architectural decisions, coding conventions |
| `DECISION` | Why we chose X over Y |
| `FEATURE` | New feature implementations |
| `ERROR` | Common errors and how to resolve them |
| `ARCHITECTURE` | System design patterns |
| `WORKFLOW` | Process/workflow documentation |

### Usage Examples

```bash
# After fixing a bug
node scripts/mem-log.js FIX "Auth Token Bug" "Fixed JWT expiry by adding timezone normalization"

# Recording a pattern
node scripts/mem-log.js PATTERN "Database Access" "All DB queries go through repository layer"

# Recording a decision
node scripts/mem-log.js DECISION "State Management" "Using React Context instead of Redux for simplicity"

# Recalling context before a task
node scripts/mem-recall.js "authentication"
node scripts/mem-recall.js "expired items"
```

## Memvid Memory Protocol

### REQUIRED: Before Starting Work
Run `node scripts/mem-recall.js "<task keywords>"` to check for relevant project context.

### REQUIRED: Automatic Storage Triggers
Store memories using `node scripts/mem-log.js` on ANY of:
- **Bug fix** → problem + solution
- **Architecture decision** → choice + rationale  
- **Pattern discovered** → reusable approach
- **Feature completed** → what was built and why
- **Error resolved** → error message + fix

### Search Tips
- Lexical search is **case-aware** — capitalize key terms (e.g., "Controllers" not "controllers")
- Use specific terms from your codebase
- Check `memvid timeline project-memory.mv2` to see all stored memories

Do NOT wait to be asked. Memory storage should happen as you work.  


## 9. Security Code Review

### Security Checklist

Before APPROVAL, verify:

-   ✅ No hardcoded secrets (API keys, passwords, tokens)
-   ✅ All user input validated (using validation middleware or libraries like `joi`, `zod`)
-   ✅ All user input sanitized when displayed
-   ✅ No SQL injection (use parameterized queries or ORM)
-   ✅ Authentication required on sensitive endpoints
-   ✅ Authorization verified (current user can perform action?)
-   ✅ Sensitive data logged appropriately (no passwords in logs)
-   ✅ CSRF protection enabled (if applicable)
-   ✅ Rate limiting on public endpoints (if applicable)

**Good:**

```typescript 
// src/controllers/usersController.ts  
import { Request, Response } from 'express';  
import { usersService } from '../services/usersService';  
import { authenticateUser, authorizeAdmin } from '../middleware/auth';  
  
export const usersController = {  
  async update(req: Request, res: Response) {  
    try {  
      // Validate input  
      const validated = updateUserSchema.parse(req.body);  
        
      // Authorization check  
      if (req.user.id !== Number(req.params.id) && !req.user.isAdmin) {  
        return res.status(403).json({ error: 'Not authorized' });  
      }  
        
      const user = await usersService.update(Number(req.params.id), validated);  
      res.json(user);  
    } catch (error) {  
      res.status(400).json({ error: error.message });  
    }  
  }  
};  
  
// src/routes/users.ts  
router.put('/:id', authenticateUser, usersController.update);
```

**Anti-Pattern:**

```
// DONT DO THIS  
export const usersController = {  
  async update(req: Request, res: Response) {  
    // Raw SQL - vulnerable to injection!  
    const sql = `UPDATE users SET email = '${req.body.email}' WHERE id = ${req.params.id}`;  
    await db.run(sql);  
    res.json({ success: true });  
  }  
};
```

### Code Review Gates

Must pass before merge:

1.  ✅ Tests passing (`npm test`)
2.  ✅ Linter clean (`npm run lint`)
3.  ✅ UBS passed (`ubs <changed-files>` exit 0)
4.  ✅ Security checklist passed
5.  ✅ No hardcoded secrets
6.  ✅ Follows Express/TypeScript conventions
7.  ✅ Comments explain why, not what
8.  ✅ OpenSpec validation passed (`openspec validate --all`)

----------

## 10. Troubleshooting

### Agent Stuck Protocol

**Condition:** Same error three consecutive attempts

**Response:**

1.  **Diagnose**: Root cause of error, not symptom
2.  **Load more context**: Relevant controllers, services, existing patterns
3.  **Propose alternative**: Different technical approach
4.  **Request help**: Ask user for clarification or direction

### Common Issues

| Problem | Symptom | Solution |
|---------|---------|----------|
| **Test Failures** | `npm test` returns non-zero | Read error message, check test expectations, verify test data setup/mocks |
| **Linter Errors** | `npm run lint` fails | Run linter with auto-fix, resolve remaining manually |
| **UBS False Positives** | UBS flags something that's not a bug | Check context, ignore if safe, document why in code comment |
| **Database Issues** | Connection or query errors | Check SQLite3 connection string, verify migrations run, check file permissions |
| **TypeScript Errors** | `tsc` compilation fails | Check types, imports, and `tsconfig.json` settings |
| **Dependency Issues** | Package not found error | Run `npm install` or `yarn install`, verify `package.json` and `package-lock.json` committed |
| **State Issues** | Tests pass individually, fail together | Check test isolation, use `beforeEach`/`afterEach` hooks, avoid shared state |
| **Performance** | Endpoint slow | Profile with logging, check N+1 queries, add database indexes |
| **OpenSpec Validation** | `openspec validate` fails | Check delta spec format, ensure scenarios use `####` headers, verify SHALL/MUST in requirements |

----------

## Best Practices Summary

### DO:

-   ✅ Follow Express REST patterns (routes → controllers → services → data access)
-   ✅ Use TypeScript strict mode with explicit types
-   ✅ Write tests before code (TDD)
-   ✅ Inject dependencies for testability
-   ✅ Keep controllers thin, logic in services
-   ✅ Use parameterized queries (avoid raw SQL)
-   ✅ Validate input, sanitize output
-   ✅ Comment why, not what
-   ✅ Cite code in plans (`src/controllers/users.ts:42`)
-   ✅ Scan with UBS before commit (catches bugs early)
-   ✅ Use OpenSpec for change tracking (not markdown TODOs)
-   ✅ Request approval before merge
-   ✅ Document architectural decisions
-   ✅ Load `packages.md` when working with packages
-   ✅ Follow CRA conventions for React (components in `src/`, standard folder structure)
-   ✅ Validate OpenSpec changes before implementation

### DON'T:

-   ❌ Hardcode secrets, APIs, or config
-   ❌ Skip tests or commit untested code
-   ❌ Skip UBS scanning (bugs found early ≠ bugs found in prod)
-   ❌ Modify code without reading full context
-   ❌ Use mock data in production
-   ❌ Write raw SQL instead of using ORM/query builder properly
-   ❌ Put business logic in controllers
-   ❌ Use `any` type without justification
-   ❌ Introduce unjustified abstraction
-   ❌ Commit to `main`/`master` directly
-   ❌ Force-push to shared branches
-   ❌ Ignore security checklist
-   ❌ Create markdown TODOs instead of OpenSpec changes
-   ❌ Skip OpenSpec validation steps
-   ❌ Eject CRA unless absolutely necessary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jatwell93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
