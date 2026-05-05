## cobalt

> > **💡 Collaboration First**: This project values collaborative decision-making. When facing decisions about architecture, developer experience, or anything with significant impact, **always present options and wait for user input** before proceeding. See [Decision-Making Philosophy](#decision-making-philosophy) for details.

# CLAUDE.md — Development Guide

> **💡 Collaboration First**: This project values collaborative decision-making. When facing decisions about architecture, developer experience, or anything with significant impact, **always present options and wait for user input** before proceeding. See [Decision-Making Philosophy](#decision-making-philosophy) for details.

## Project Overview

Cobalt is a TypeScript CLI testing framework for AI agents and LLM-powered applications. Think "Unit testing for AI Agents" — it provides experiment runners, evaluators, datasets, and result tracking.

### Tech Stack

- **Language**: TypeScript 5.7 with strict mode
- **Build Tool**: tsup for bundling
- **CLI Framework**: citty for command-line interface
- **Testing**: Vitest for unit and integration tests
- **Code Quality**: Biome for linting and formatting
- **Database**: better-sqlite3 for history tracking
- **HTTP Server**: Hono for dashboard API
- **Package Manager**: pnpm

## Architecture

Cobalt follows a clean, modular architecture:

```
packages/cobalt/
├── src/
│   ├── core/               # Core experiment runner logic
│   │   ├── experiment.ts   # Main experiment() function
│   │   ├── Evaluator.ts    # Evaluator class
│   │   └── config.ts       # Configuration system
│   ├── datasets/           # Dataset loading and transformation
│   │   └── Dataset.ts      # Dataset class
│   ├── evaluators/         # Evaluator implementations
│   │   ├── llm-judge.ts    # LLM-based evaluation
│   │   ├── function.ts     # Custom function evaluation
│   │   ├── exact-match.ts  # String matching
│   │   └── similarity.ts   # Embeddings (P2 - not implemented)
│   ├── cli/                # CLI commands
│   │   ├── index.ts        # CLI entry point
│   │   └── commands/       # Individual commands (run, init, etc.)
│   ├── dashboard/          # Dashboard server (P4)
│   │   ├── server.ts       # Hono server
│   │   └── routes.ts       # API routes
│   ├── mcp/                # Model Context Protocol (P3)
│   │   ├── server.ts       # MCP server
│   │   └── tools.ts        # MCP tool implementations
│   ├── storage/            # Data persistence
│   │   ├── results.ts      # JSON result files
│   │   ├── cache.ts        # LLM response cache
│   │   └── db.ts           # SQLite history database
│   ├── utils/              # Utilities
│   │   ├── cost.ts         # Token cost estimation
│   │   ├── stats.ts        # Statistical calculations
│   │   ├── template.ts     # Template rendering
│   │   └── hash.ts         # Hash generation
│   └── types/              # TypeScript types
└── tests/                  # Test suite
    ├── unit/               # Unit tests
    ├── integration/        # Integration tests
    └── helpers/            # Test helpers and mocks
```

## Development Guidelines

### 1. Adding New Features

When adding a feature, follow these steps:

#### For New Evaluator Types (e.g., similarity)

1. **Create evaluator implementation** in `src/evaluators/`
2. **Define types** in `src/types/index.ts`
3. **Update Evaluator class** dispatch logic in `src/core/Evaluator.ts`
4. **Write unit tests** in `tests/unit/evaluators/`
5. **Update documentation** in `.memory/documentation.md`

#### For New CLI Commands

1. **Create command file** in `src/cli/commands/`
2. **Register command** in `src/cli/index.ts`
3. **Write tests** (if feasible with mocked file system)
4. **Update README** with command usage

#### For New Dataset Loaders

1. **Add loader method** to `src/datasets/Dataset.ts`
2. **Write unit tests** in `tests/unit/Dataset.test.ts`
3. **Update documentation**

### 2. Writing Tests

**Unit tests are mandatory** for all core functionality (experiment runner, evaluators, datasets, utilities).

**Test Structure:**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'

describe('FeatureName', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('should do something', async () => {
    // Arrange
    const input = 'test'
    
    // Act
    const result = await functionUnderTest(input)
    
    // Assert
    expect(result).toBe('expected')
  })
})
```

**Mocking Guidelines:**
- **LLM APIs**: Mock `openai` and `@anthropic-ai/sdk` modules
- **File system**: Mock `node:fs` or use helper functions
- **External services**: Always mock, never make real API calls in tests

**Run Tests:**
```bash
pnpm test              # Run all tests
pnpm test:watch        # Watch mode
pnpm test:coverage     # With coverage report
```

### 3. Code Quality

**Before Committing (Required):**
```bash
pnpm test              # Run tests - MUST PASS
pnpm lint              # Check linting - MUST PASS
pnpm build             # Verify build works - MUST PASS
```

**Optional (but recommended):**
```bash
pnpm format            # Format code
pnpm check             # Auto-fix issues
```

**Code Standards:**
- Use TypeScript strict mode - no `any` types
- Write descriptive variable and function names
- Add comments only when logic isn't self-evident
- Keep functions small and focused
- Handle errors explicitly
- Use functional programming patterns where appropriate

### 4. Git Workflow

**IMPORTANT: Always create a new branch for each new feature or Claude session, verify tests/linting pass, commit, and create a PR.**

#### Branching Strategy

**Create a new branch when:**
- Starting a new Claude session
- Working on a new feature or fix
- Addressing a different issue or task

**Format:** `category/short-description`

#### Standard Workflow

1. **Create a new branch** for your changes:
```bash
git checkout -b category/short-description
# Examples:
# - feat/similarity-evaluator
# - fix/dataset-csv-parsing
# - docs/api-updates
# - ci/github-actions-setup
```

2. **Make your changes** to the code

3. **Verify quality checks pass** before committing:
```bash
# Run tests
pnpm test

# Check linting
pnpm lint

# Optional: run full check
pnpm check && pnpm build
```

4. **Stage and commit** with a concise message:
```bash
git add .
git commit -m "type: short description (few words)"
```

5. **Rebase on main before pushing**:
```bash
# Fetch latest changes
git fetch origin

# Rebase your branch on main
git rebase origin/main

# If conflicts occur, resolve them and continue
git rebase --continue
```

6. **Push and create PR**:
```bash
# Push to remote (use -f if you rebased)
git push -u origin your-branch-name

# Create PR with gh CLI
gh pr create --title "Short PR title" --body "PR description" --base main
```

#### Adding to Existing PR (Same Claude Session Only)

**Only if you're continuing work in the SAME Claude session** on the SAME feature:

1. **Stay on the same branch** (don't create a new one)
2. **Make your additional changes**
3. **Verify quality checks pass**:
```bash
pnpm test && pnpm lint
```
4. **Commit with a descriptive message**:
```bash
git add .
git commit -m "type: short description"
```
5. **Rebase on main and push**:
```bash
git fetch origin
git rebase origin/main
git push  # Updates existing PR automatically
```

**When to add to same PR vs. create new branch:**
- ✅ Same branch: Continuing work in the SAME Claude session on the SAME feature
- ❌ New branch: New Claude session, different feature, unrelated changes

#### Commit Message Guidelines

**Keep commit messages SHORT (few words only):**

✅ **Good examples:**
```
feat: add similarity evaluator
fix: CSV parsing bug
docs: update README badges
test: add evaluator tests
refactor: simplify dispatch logic
chore: upgrade dependencies
```

❌ **Bad examples (too long):**
```
feat: add similarity evaluator with embeddings support and comprehensive tests
fix: resolve the dataset CSV parsing bug that was causing issues with commas
docs: update the API documentation and add examples for all evaluators
```

**Conventional commit types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `test:` - Test additions/changes
- `refactor:` - Code refactoring
- `style:` - Code style/formatting
- `chore:` - Maintenance tasks
- `ci:` - CI/CD changes

#### Pre-Commit Checklist

Before every commit, ensure:
- ✅ Tests pass (`pnpm test`)
- ✅ Linting passes (`pnpm lint`)
- ✅ Build succeeds (`pnpm build`)
- ✅ Commit message is concise (few words)
- ✅ Changes are on a feature branch (not `main`)

#### Pre-Push Checklist

Before pushing to remote, ensure:
- ✅ All commits are on a feature branch
- ✅ Branch is rebased on latest `main` (`git rebase origin/main`)
- ✅ All tests still pass after rebase
- ✅ No merge conflicts exist

#### Never Commit/Push:

- ❌ Broken tests
- ❌ Linting errors
- ❌ Build failures
- ❌ Directly to `main` branch
- ❌ Without running quality checks
- ❌ Without rebasing on main first

### 5. Error Handling

**For Core Functionality:**
- Use try-catch blocks appropriately
- Return meaningful error messages
- For evaluators: Return `{ score: 0, reason: "error message" }` instead of throwing

**Example:**
```typescript
try {
  const result = await llmCall()
  return parseResult(result)
} catch (error) {
  return {
    score: 0,
    reason: `Evaluation error: ${error instanceof Error ? error.message : 'Unknown error'}`
  }
}
```

### 6. Evaluator Implementation

When implementing a new evaluator:

**Required signature:**
```typescript
export async function evaluateMyType(
  config: MyTypeEvaluatorConfig,
  context: EvaluationContext,
  apiKey?: string
): Promise<EvaluationResult> {
  // Implementation
  return {
    score: 0.85,
    reason: 'Explanation of score'
  }
}
```

**Validation:**
- Score must be between 0 and 1
- Provide clear reasoning for the score
- Handle errors gracefully

### 7. Dataset Transformations

Dataset methods should be **chainable** and **immutable**:

```typescript
class Dataset {
  map(fn: (item: ExperimentItem) => ExperimentItem): Dataset {
    return new Dataset({ items: this.items.map(fn) })
  }
  
  filter(predicate: (item: ExperimentItem) => boolean): Dataset {
    return new Dataset({ items: this.items.filter(predicate) })
  }
}
```

## Memory System

**Always maintain the `.memory/` folder** with up-to-date information:

- **decisions.md**: Document architectural choices and reasoning
- **analysis.md**: Keep codebase structure and component documentation current
- **progress.md**: Log significant milestones and completed work
- **documentation.md**: Keep API documentation comprehensive
- **roadmap.md**: Update with future plans and next steps

**When to Update:**
- After adding new features
- When making architectural decisions
- When encountering and solving problems
- After significant refactoring
- When questions arise that need clarification

## Common Commands

### Development
```bash
pnpm dev                    # Watch mode for development
pnpm build                  # Build for production
pnpm build:watch            # Watch build
```

### Testing & Quality
```bash
pnpm test                  # Run all tests
pnpm test:watch            # Run tests in watch mode
pnpm test:coverage         # Run tests with coverage
pnpm lint                  # Check code quality
pnpm format                # Format code
pnpm check                 # Lint and format
```

### CLI Commands
```bash
pnpm cobalt run <file>     # Run an experiment
pnpm cobalt init           # Initialize new project
pnpm cobalt history        # View past runs
pnpm cobalt compare <id1> <id2>  # Compare runs
pnpm cobalt serve          # Start dashboard
pnpm cobalt clean          # Clean cache/results
pnpm cobalt mcp            # Start MCP server
```

## Decision-Making Philosophy

**IMPORTANT: When in doubt, ask the user.** This is a collaborative project. The user values being involved in decisions that impact the project's direction, architecture, and developer experience.

### Always Ask About:

#### 🏗️ Architecture & Design
- Adding new evaluator types or patterns
- Changing how evaluators are dispatched
- Introducing new dependencies or frameworks
- Modifying the storage layer (SQLite, file structure)
- API design decisions (experiment options, Dataset methods)
- Adding new CLI commands

#### 🛠️ Developer Experience
- Build tooling changes (tsup, bundlers)
- Testing strategy changes
- CLI argument parsing or command structure
- Configuration format changes

#### 📦 Dependencies & Infrastructure
- Adding major dependencies
- LLM provider integrations (OpenAI, Anthropic, others)
- Storage backend changes
- Dashboard framework choices

#### 🎨 User-Facing Impact
- CLI output format changes
- Result file structure modifications
- Breaking API changes
- Configuration schema changes

### Decision-Making Process

For **decisions that require approval**:

1. **Identify** that a decision point has been reached
2. **Research** 2-3 viable options
3. **Present** options clearly:
   ```
   I need to make a decision about [X]. Here are the options:

   Option 1: [Name]
   - Pros: ...
   - Cons: ...
   - Context: ...

   Option 2: [Name]
   - Pros: ...
   - Cons: ...
   - Context: ...

   My recommendation: [X] because [reasoning]

   What would you prefer?
   ```
4. **Wait for approval**
5. **Document** the decision in `.memory/decisions.md`
6. **Implement** following the approved approach

### When You Can Proceed Autonomously

- Following existing patterns in the codebase
- Writing tests that follow existing test patterns
- Fixing obvious bugs or typos
- Adding documentation
- Formatting and linting fixes
- Implementing already-decided features

## Best Practices

### DO:
✅ **Ask before making impactful decisions**
✅ Write tests for all core functionality
✅ Use strict TypeScript
✅ Keep modules focused and cohesive
✅ Document decisions and changes
✅ Handle errors gracefully
✅ Make evaluators return errors as `{score: 0}` not throw
✅ Keep CLI commands simple and composable
✅ Cache LLM responses to save costs
✅ Track token usage and costs

### DON'T:
❌ Make architectural decisions without user approval
❌ Skip writing tests for core features
❌ Use `any` type
❌ Make breaking API changes without discussion
❌ Add dependencies without consideration
❌ Throw errors from evaluators (return {score: 0} instead)
❌ Make real API calls in tests
❌ Commit broken code

## Testing Philosophy

**Core Principle**: Test behavior, not implementation.

**What to Test:**
- ✅ Public API functions (experiment, Evaluator, Dataset)
- ✅ Evaluator implementations
- ✅ Dataset transformations
- ✅ Utility functions (cost, stats, template)
- ✅ Error handling and edge cases

**What NOT to Test:**
- ❌ Private implementation details
- ❌ Third-party library behavior
- ❌ Simple getters/setters

**Current Coverage**: 231 tests across 12 test suites covering P0-P3 features (80-100% for tested modules)

## Getting Help

- **README.md**: User-facing documentation
- **INSTRUCTIONS.md**: Complete specification and requirements
- **.memory/**: Project context, decisions, and progress
- **Architecture Questions**: Review existing patterns in similar features
- **Stuck?**: Ask the user for clarification or guidance

## Next Steps

See `.memory/roadmap.md` for planned features and remaining work.

**Current Status**: P0-P3 are 95% complete (P0: MVP ✅, P1: Usable ✅, P2: Powerful ✅, P3: Connected ✅)

**Completed in P2-P3:**
- ✅ CI mode with quality thresholds
- ✅ Plugin system for custom evaluators
- ✅ Autoevals integration (11 evaluator types)
- ✅ Complete MCP implementation (4 tools, 3 resources, 3 prompts)
- ✅ Statistical aggregations (avg, min, max, p50, p95, passRate)
- ✅ Auto-generate experiments from agent code

**Remaining P3/P4 work:**
- [ ] Dashboard frontend UI (backend API complete)
- [ ] Remote dataset loaders (Dataset.fromRemote)
- [ ] Similarity evaluator with embeddings
- [ ] Multiple runs with statistical aggregation

---
> Source: [basalt-ai/cobalt](https://github.com/basalt-ai/cobalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
