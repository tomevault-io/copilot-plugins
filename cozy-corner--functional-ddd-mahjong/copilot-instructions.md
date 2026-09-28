## functional-ddd-mahjong

> Mahjong Functional DDD Learning Project - Development workflow and constraints for Claude Code.

# Claude Context

## Project Overview
Mahjong Functional DDD Learning Project - Development workflow and constraints for Claude Code.

## ⚠️ CRITICAL: Git Operations - User Permission Required

### Git Command Restrictions
**NEVER execute git commands without explicit user instruction:**

- `git add`
- `git commit` 
- `git push`
- `git merge`
- `git rebase`
- `git reset`
- `git revert`

### Required Workflow
1. **Make code changes** as requested
2. **Run tests** to verify changes work
3. **ASK USER** before any git operations: "Ready to commit these changes?"
4. **WAIT** for explicit user approval before proceeding
5. **Only then** execute git commands if user approves

### Exception: Explicit User Commands
Git operations are ONLY allowed when user explicitly requests them with commands like:
- "commit these changes"
- "create a PR"
- "add and commit"
- "push to remote"

### TodoWrite Rules
- **NEVER** add "commit changes" or similar git tasks to TodoWrite
- Focus TodoWrite on code implementation tasks only
- Let user decide when to commit

## Development Workflow

### Reference Documents
- **Learning content**: See project README.md and todo.md  
- **This document**: Development constraints and workflow only

## Development Guidelines

### Code Quality Rules
**MANDATORY**: Run checks before any commit:
- After implementing code: `make test`
- **Before final commit**: `make check` (runs build, test, and lint)
- Use project-specific formatting and linting tools

Available commands:
- `make build` - Build the project
- `make test` - Run all tests
- `make lint` - Check code formatting
- `make format` - Auto-format code
- `make check` - Run all pre-commit checks
- `make watch` - Run tests in watch mode
- `make clean` - Clean build artifacts

**Workflow**:
1. Development: `make build && make test`
2. Before commit: `make check` (ensures build, tests, and formatting pass)
3. Fix formatting issues: `make format` if lint fails

## ⚠️ CRITICAL: Code Quality Policy

### 🚫 STRICTLY FORBIDDEN:
- **NEVER bypass quality checks** under any circumstances
- **NEVER commit failing tests**
- **NEVER commit code that doesn't build**
### ✅ REQUIRED before any commit:
1. **CHECK**: Run `make check` - this will:
   - Build the project
   - Run all tests
   - Verify code formatting
2. **FIX**: If formatting fails, run `make format`
3. **COMMIT**: Only after `make check` passes successfully

**Remember**: Manual quality checks are the final gate. Skipping them compromises the entire codebase integrity.

## Testing Guidelines
**Use project's testing framework for all tests**

**Running tests**:
- `make test` - Run all tests
- `make watch` - Run tests in watch mode
- `dotnet test --filter "TestClassName"` - Run specific test class

## ⚠️ CRITICAL: F# Project File Management

### F# File Addition Rules
**NEVER forget to add new .fs files to .fsproj**

**❌ COMMON MISTAKE - Creating files without adding to project**:
```bash
# Creating Meld.fs and Pair.fs but forgetting to add them to:
# - src/FunctionalDddMahjong.Domain/FunctionalDddMahjong.Domain.fsproj
# - tests/FunctionalDddMahjong.Domain.Tests/FunctionalDddMahjong.Domain.Tests.fsproj
```

**✅ REQUIRED WORKFLOW**:
1. **Create .fs file**
2. **IMMEDIATELY add to appropriate .fsproj file**
3. **Respect dependency order**: `Tile.fs` → `Meld.fs, Pair.fs` → `Hand.fs`
4. **Verify with**: `dotnet build`

**Why**: F# requires explicit file inclusion and dependency order unlike C#

## Functional DDD Development Focus

### Code Quality Principles
- **Type Safety First**: Eliminate invalid states at compile time
- **Pure Functions**: Minimize side effects, maximize testability  
- **Function Composition**: Build complex logic from small, composable functions
- **Clear Error Handling**: Use Result types and Railway-Oriented Programming

## ⚠️ CRITICAL: Console Output Guidelines

### Console Output Best Practices
For this learning project, minimal console output is acceptable for debugging purposes:

**✅ ACCEPTABLE - Learning and debugging**:
```
Debug: Analyzing hand
Result: Found yaku
```

**Guidelines**:
- Keep debug output minimal and meaningful
- Remove debug prints before final commits
- Focus on domain logic rather than output formatting

## ⚠️ CRITICAL: Git Branch Management Rules

### Branch Creation Policy
**NEVER create branches without explicit user instruction**

**❌ FORBIDDEN - Creating branches autonomously**:
```bash
git checkout -b feat/tile-implementation    # DON'T create branches automatically
```

**✅ CORRECT - Ask user before creating branches**:
```bash
# 1. Ask user: "Should I create a new branch for this phase?"
# 2. Wait for explicit approval
# 3. Only then create branch with user-provided name
```

**If branch creation is needed**:
- **ASK FIRST**: "Ready to create a branch? What should it be named?"
- **WAIT for approval** before executing git commands
- **Use the exact name** provided by user

**Why**: Autonomous branch creation leads to scattered work, lost changes, and workflow confusion.

### Git Stash Safety Rules
**Always verify stash contents before applying**

**❌ DANGEROUS - Blind stash operations**:
```bash
git stash pop    # DON'T pop without checking contents
```

**✅ SAFE - Verify before applying**:
```bash
git stash list                    # Check what stashes exist
git stash show stash@{0}          # See which files are affected
git stash show stash@{0} --stat   # See change summary
# Only then decide if you want to apply it
```

**Why**: Stash may contain unrelated changes from previous experiments, causing unexpected conflicts.

## Learning Documentation
- Main learning guide: `README.md`
- Learning progress TODO: `todo.md`
- Phase-specific notes should be documented in commit messages
- Keep track of functional DDD concepts learned in each phase

## ⚠️ CRITICAL: PR Review Comment Check (MUST DO FIRST)
When working on PRs, **ALWAYS execute these two commands FIRST**:

```bash
# Command 1: Get PR overview
gh pr view {pr_number} --comments

# Command 2: Get line-by-line comments (CRITICAL - often missed)
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

**WARNING**: Skipping Command 2 will cause you to miss important code-level feedback and bugs.

## PR Comment Response Rules

### Required Steps (Execute in this order)
1. **Get PR review comments**: `gh api repos/{owner}/{repo}/pulls/{pr_number}/comments`
2. **Report all comments to user**: List all comments with their IDs and content
3. **Wait for user instructions**: DO NOT reply to any comments without explicit user approval
4. **Reply only when instructed**: 
   ```bash
   gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --method POST \
     --field body="User-provided reply message" \
     --field in_reply_to={comment_id}
   ```

### ⚠️ CRITICAL: Never Auto-Reply
- **NEVER** reply to comments without explicit user instruction
- **ALWAYS** ask user how to respond to each comment
- **WAIT** for user to provide specific reply text or approve "won't implement" responses

### ⚠️ FORBIDDEN Commands (Never Use)
Do not use these commands (Reason: Posts to PR body or overwrites existing comments)
- `gh pr comment {pr_number} --body="..."`
- `gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --method POST --field body="..."` (without in_reply_to)
- `gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --method POST` (overwrites existing comment)

### Response Requirements (When User Instructs)
1. **Reply to EVERY comment** user requests response to
2. **For fixes**: 
   - Implement the fix
   - Commit with descriptive message
   - Push to remote branch
   - Include commit IDs in reply: `Fixed in commit [abc123](link)`
3. **For non-fixes**: Mark status clearly as "Won't implement" or "Acknowledged" (only when user specifies)
4. **Use in_reply_to** for direct comment replies

---
> Source: [cozy-corner/functional-ddd-mahjong](https://github.com/cozy-corner/functional-ddd-mahjong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
