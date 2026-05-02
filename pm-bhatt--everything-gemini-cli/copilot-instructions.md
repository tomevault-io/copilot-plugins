## everything-gemini-cli

> This extension provides comprehensive development workflows, best practices, and commands for Gemini CLI.

# Everything Gemini CLI - Development Guidelines

This extension provides comprehensive development workflows, best practices, and commands for Gemini CLI.

## Core Principles

### 1. Security First

Before ANY commit:
- No hardcoded secrets (API keys, passwords, tokens)
- All user inputs validated
- SQL injection prevention (parameterized queries)
- XSS prevention (sanitized HTML)
- CSRF protection enabled
- Authentication/authorization verified
- Rate limiting on all endpoints
- Error messages don't leak sensitive data

**Secret Management:**
```typescript
// NEVER: Hardcoded secrets
const apiKey = "sk-proj-xxxxx"

// ALWAYS: Environment variables
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

**Security Response Protocol:**
If security issue found:
1. STOP immediately
2. Run `/security-review` command
3. Fix CRITICAL issues before continuing
4. Rotate any exposed secrets
5. Review entire codebase for similar issues

### 2. Coding Style

**Immutability (CRITICAL):**
ALWAYS create new objects, NEVER mutate:

```javascript
// WRONG: Mutation
function updateUser(user, name) {
  user.name = name  // MUTATION!
  return user
}

// CORRECT: Immutability
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

**File Organization:**
MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large components
- Organize by feature/domain, not by type

**Error Handling:**
ALWAYS handle errors comprehensively:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('Detailed user-friendly message')
}
```

**Input Validation:**
ALWAYS validate user input:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

**Code Quality Checklist:**
Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No console.log statements
- [ ] No hardcoded values
- [ ] No mutation (immutable patterns used)

### 3. Testing Requirements

**Minimum Test Coverage: 80%**

Test Types (ALL required):
1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations
3. **E2E Tests** - Critical user flows (Playwright)

**Test-Driven Development:**
MANDATORY workflow:
1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)

### 4. Git Workflow

**Commit Message Format:**
```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

**Pull Request Workflow:**
When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

**Feature Implementation Workflow:**
1. **Plan First** - Use `/plan` command to create implementation plan
2. **TDD Approach** - Use `/tdd` command
3. **Code Review** - Use `/code-review` command immediately after writing code
4. **Commit & Push** - Detailed commit messages

### 5. Performance Optimization

**Context Window Management:**
Avoid last 20% of context window for:
- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions

Lower context sensitivity tasks:
- Single-file edits
- Independent utility creation
- Documentation updates
- Simple bug fixes

**Build Troubleshooting:**
If build fails:
1. Use `/build-fix` command
2. Analyze error messages
3. Fix incrementally
4. Verify after each fix

### 6. Common Patterns

**API Response Format:**
```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

**Custom Hooks Pattern:**
```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

**Repository Pattern:**
```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## Available Commands

Use `/command-name` to invoke these commands:

| Command | Description |
|---------|-------------|
| `/plan` | Create implementation plan before coding |
| `/tdd` | Enforce test-driven development workflow |
| `/code-review` | Comprehensive code review |
| `/build-fix` | Fix TypeScript and build errors |
| `/e2e` | Generate and run E2E tests |
| `/refactor-clean` | Safely remove dead code |
| `/checkpoint` | Create/verify workflow checkpoints |
| `/verify` | Run comprehensive verification |
| `/learn` | Extract reusable patterns |
| `/eval` | Manage eval-driven development |
| `/orchestrate` | Sequential multi-step workflows |
| `/test-coverage` | Analyze and improve test coverage |
| `/update-docs` | Sync documentation from source |
| `/update-codemaps` | Update architecture documentation |
| `/setup-pm` | Configure package manager |

## Available Skills

Skills are activated automatically based on context. Load with `/skills <skill-name>`:

| Skill | When to Use |
|-------|-------------|
| `tdd-workflow` | Writing new features, fixing bugs |
| `coding-standards` | All development work |
| `backend-patterns` | API and server-side development |
| `frontend-patterns` | React/Next.js development |
| `postgres-patterns` | Database queries and schemas |
| `clickhouse-io` | Analytics and data engineering |
| `security-review` | Auth, user input, sensitive data |
| `iterative-retrieval` | Complex context retrieval |
| `eval-harness` | Evaluation-driven development |
| `continuous-learning` | Extract patterns from sessions |

## Pre-Commit Checklist

Before every commit, verify:

1. **Build Passes**
   ```bash
   npm run build  # or pnpm build
   ```

2. **Tests Pass**
   ```bash
   npm test
   ```

3. **No Secrets Exposed**
   - Check for hardcoded API keys
   - Verify .env files are gitignored

4. **Code Quality**
   - No console.log statements
   - Functions < 50 lines
   - Files < 800 lines

5. **Coverage > 80%**
   ```bash
   npm run test:coverage
   ```

## Hooks (Manual Checklist)

Since Gemini CLI has limited hook support, manually check:

**Before Long-Running Commands:**
- Consider running in tmux for session persistence
- Use: `tmux new -s dev` | `tmux attach -t dev`

**Before Git Push:**
- Review all changes
- Run full test suite
- Check for console.log statements

**After Editing Code:**
- Run Prettier: `npx prettier --write <file>`
- Run TypeScript check: `npx tsc --noEmit`
- Check for console.log in modified files

## Feature Parity Notes

Features available in Claude Code but not in Gemini CLI:
- Agent-specific tool restrictions (inlined into command prompts)
- Model selection per command (opus/haiku/sonnet)
- PreToolUse/PostToolUse hooks (documented as manual steps)
- Auto-formatting on edit (run manually)

---

*Adapted from everything-claude-code for Gemini CLI*

---
> Source: [pm-bhatt/everything-gemini-cli](https://github.com/pm-bhatt/everything-gemini-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
