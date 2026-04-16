## apicurio-registry-vscode-plugin-poc

> This file contains project-specific instructions for AI assistants working on the Apicurio Registry VSCode Extension.

# Apicurio VSCode Plugin - Claude Code Instructions

This file contains project-specific instructions for AI assistants working on the Apicurio Registry VSCode Extension.

## Project Overview

**What**: VSCode extension for browsing and editing API specifications from Apicurio Registry
**Tech Stack**: TypeScript, VSCode Extension API, Axios, Model Context Protocol (MCP)
**Current Phase**: Active development - implementing core features

## Documentation Structure

### Critical Documentation Files

1. **`docs/TODO.md`** - Daily quick reference (what to work on today)
2. **`docs/MASTER_PLAN.md`** - Strategic overview and milestones
3. **`docs/tasks/[status]/[priority]/XXX-name.md`** - Individual task specifications
4. **`docs/ai-integration/`** - AI/MCP integration planning and testing

### Documentation Locations

- **Project root**: `/apicurio-vscode-plugin/`
- **Source code**: `src/`
- **Tests**: `test/` or `src/test/suite/`
- **Documentation**: `docs/`
- **Planning**: `docs/planning/`
- **AI Integration**: `docs/ai-integration/`
- **Testing guides**: `docs/testing/`
- **Test data**: `test-data/`

## CRITICAL RULE: Documentation Management

**Documentation is NOT optional. It's part of completing the task.**

### Always Update Documentation When:

1. ✅ Completing a task
2. ✅ Starting a new task
3. ✅ Making architectural decisions
4. ✅ Discovering issues or blockers
5. ✅ Changing priorities
6. ✅ Weekly reviews

### Documentation Update Workflow

#### At Task Completion - MANDATORY

**STOP and ask user:** "Task XXX complete. Let's update documentation together."

**Update checklist:**
- [ ] Move task file: `tasks/in-progress/` → `tasks/completed/`
- [ ] Add "Lessons Learned" section to task file
- [ ] Update `docs/TODO.md`:
  - Move to "Recently Completed" table
  - Update progress percentages
  - Update "What to Work on TODAY"
  - Add entry to "Recent Activity Log"
- [ ] Update `docs/MASTER_PLAN.md` if milestone reached
- [ ] Confirm all updates before proceeding to next task

#### At Task Start

**Ask user:** "Starting Task XXX. Should we update TODO.md?"

**Actions:**
- Update TODO.md "What to Work on TODAY"
- Move task file to `in-progress/` if needed
- Update progress bars

### Quality Check Before Task Completion

**Task is NOT complete until:**
- [ ] Task file moved to completed/
- [ ] TODO.md updated with completion
- [ ] Progress percentages recalculated
- [ ] Activity log entry added
- [ ] MASTER_PLAN.md updated if milestone reached
- [ ] Lessons learned documented
- [ ] User confirms all updates

## Git Workflow - MANDATORY

### Branch Strategy

**ALWAYS use feature branches. NEVER commit directly to main.**

#### Branch Naming Convention

```
task/XXX-short-description
```

**Examples:**
- `task/003-context-menus`
- `task/004-add-version`
- `task/007-delete-operations`

**For non-task work:**
- `fix/bug-description` - Bug fixes
- `docs/documentation-update` - Documentation only
- `refactor/component-name` - Code refactoring
- `test/feature-name` - Test additions

### Workflow for Every Task

**1. Before Starting:**
```bash
git checkout main
git pull origin main
git checkout -b task/XXX-short-name
```

**Claude MUST ask:**
> "Starting Task XXX. First, let's create a feature branch:
> - Branch name: `task/XXX-short-description`
> - Shall I provide the git commands?"

**2. During Work:**
```bash
# Commit frequently
git add [files]
git commit -m "feat(XXX): implement [feature]"
git push -u origin task/XXX-short-name
```

**3. Before Task Completion - Run Tests:**
```bash
npm run test           # Unit tests
npm run lint          # Linting
npm run compile       # TypeScript compilation
```

**Claude MUST enforce:**
> "Before we can complete this task, we need to verify:
> 1. All tests passing
> 2. No linting errors
> 3. TypeScript compiles successfully
>
> Shall I help you run these checks?"

**4. Merge to Main (only when tests pass):**
```bash
git checkout main
git pull origin main
git merge task/XXX-short-name
npm run test && npm run compile  # Verify after merge
git push origin main
git branch -d task/XXX-short-name
```

### Git Best Practices

**DO:**
- ✅ Create branch before starting work
- ✅ Commit frequently with clear messages
- ✅ Run tests before merging
- ✅ Keep branches short-lived (1-2 days max)
- ✅ Delete branches after merging

**DON'T:**
- ❌ Commit directly to main
- ❌ Merge without running tests
- ❌ Push broken code to main
- ❌ Leave branches unmerged for weeks
- ❌ Mix multiple tasks in one branch

## Test-Driven Development (TDD) - MANDATORY

### TDD Cycle - ALWAYS Follow

```
1. RED    → Write a failing test
2. GREEN  → Write minimal code to pass
3. REFACTOR → Improve code while keeping tests green
```

**CRITICAL: Write tests BEFORE implementation code.**

### TDD Workflow

**Phase 1: RED - Write Failing Test**

```typescript
// Example: test/commands/copyCommands.test.ts
describe('copyGroupIdCommand', () => {
    it('should copy group ID to clipboard', async () => {
        const mockNode = { groupId: 'test-group' };
        await copyGroupIdCommand(mockNode);
        expect(mockClipboard.writeText).toHaveBeenCalledWith('test-group');
    });
});
```

Run test - it MUST fail:
```bash
npm run test  # Expected: FAIL ❌
```

**Phase 2: GREEN - Minimal Implementation**

```typescript
export async function copyGroupIdCommand(node: RegistryItem): Promise<void> {
    await vscode.env.clipboard.writeText(node.groupId!);
}
```

Run test - it MUST pass:
```bash
npm run test  # Expected: PASS ✅
```

**Phase 3: REFACTOR - Improve Code**

Add error handling and tests:
```typescript
it('should show error when group ID is missing', async () => {
    const mockNode = { type: RegistryItemType.Group };
    await copyGroupIdCommand(mockNode);
    expect(mockShowErrorMessage).toHaveBeenCalled();
});

export async function copyGroupIdCommand(node: RegistryItem): Promise<void> {
    if (!node.groupId) {
        vscode.window.showErrorMessage('No group ID available');
        return;
    }
    await vscode.env.clipboard.writeText(node.groupId);
    vscode.window.showInformationMessage(`Copied: ${node.groupId}`);
}
```

### Test Coverage Requirements

**Minimum coverage per task:**
- Happy path: ✅ REQUIRED
- Error cases: ✅ REQUIRED
- Edge cases: ✅ REQUIRED
- Validation: ✅ REQUIRED

**Coverage targets:**
- New code: 80%+ coverage
- Critical paths: 100% coverage

### Running Tests

```bash
npm run test              # Run all tests
npm run test -- --grep "copyCommands"  # Run specific test
npm run lint              # Linting
npm run compile           # TypeScript compilation
```

### TDD Best Practices

**DO:**
- ✅ Write test first, implementation second
- ✅ Write simplest code to pass test
- ✅ Refactor after tests pass
- ✅ Use descriptive test names
- ✅ Test edge cases and errors
- ✅ Mock external dependencies

**DON'T:**
- ❌ Skip test writing
- ❌ Write all code then test
- ❌ Leave tests failing
- ❌ Skip refactor phase
- ❌ Ignore test failures

## Definition of Done - MANDATORY

A task is NOT complete until ALL of the following are done:

### Code Requirements
- [ ] Implementation complete (all phases/features working)
- [ ] TypeScript compiles with 0 errors
- [ ] No linting errors
- [ ] All tests passing (unit tests)

### Test Documentation Requirements
- [ ] Test cases documented in task spec (tasks/*/XXX-name.md)
- [ ] Manual test guide created (tasks/*/XXX-TESTING_GUIDE.md) - Optional but recommended for complex features
- [ ] Test results recorded (pass/fail for each test case)
- [ ] Edge cases identified and tested

### Documentation Requirements
- [ ] Task spec moved to tasks/completed/
- [ ] TODO.md updated with completion
- [ ] MASTER_PLAN.md updated (if milestone reached)
- [ ] Lessons learned documented
- [ ] Recent Activity log entry added

### Git Requirements
- [ ] All changes committed to feature branch
- [ ] Tests verified passing before merge
- [ ] Branch merged to main
- [ ] Feature branch deleted

### User Confirmation
- [ ] User has reviewed and approved the implementation
- [ ] User has tested manually (or delegated testing)

**Remember:** "If documentation is not updated, the task is NOT complete."

## Commit Message Format

```
<type>: <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Example:**
```
feat: complete task 003 - context menus

- Add context menus for all tree node types
- Implement copy, open, state, download commands
- Update documentation (TODO.md, task file)

Closes #003
```

## Code Implementation Rules

### TypeScript Standards
- Always define interfaces for data structures
- Avoid 'any' type
- Use type inference where obvious
- Prefer readonly where applicable
- Maintain strict mode compliance

### Error Handling
- Use try-catch for async operations
- Provide user-friendly error messages
- Log errors for debugging
- Handle edge cases explicitly

### VSCode Extension Best Practices
- Use proper disposal for resources
- Implement cancellation tokens for long operations
- Follow VSCode UX patterns
- Test with keyboard shortcuts
- Ensure accessibility

### File Organization
- One component per file
- Keep files under 500 lines
- Group related functionality
- Use index.ts for public exports

## Task Workflow Integration

**Complete workflow (TDD + Git + Docs):**

```bash
# 1. Create feature branch
git checkout -b task/XXX-name

# 2. RED: Write failing test
# Create test file, write test cases
npm run test  # Should FAIL ❌
git add test/
git commit -m "test(XXX): add failing tests for feature"

# 3. GREEN: Implement minimal code
# Write implementation
npm run test  # Should PASS ✅
git add src/
git commit -m "feat(XXX): implement feature (tests passing)"

# 4. REFACTOR: Improve code
# Refactor, add more tests
npm run test  # Should still PASS ✅
git add .
git commit -m "refactor(XXX): improve code quality"

# 5. Create test documentation
# Add test cases to task spec (XXX-name.md)
# Create testing guide (XXX-TESTING_GUIDE.md) if complex feature
# Document test scenarios, expected results, edge cases
git add docs/tasks/
git commit -m "docs(XXX): add test documentation"

# 6. Update documentation
# Update TODO.md, task file, MASTER_PLAN.md
git add docs/
git commit -m "docs(XXX): update documentation"

# 7. Merge to main (all tests passing)
npm run test && npm run lint && npm run compile
git checkout main
git merge task/XXX-name
git push origin main
```

## Proactive Prompts

**Claude MUST proactively ask:**

- ✅ "I see we completed task XXX. Shall we update TODO.md now?"
- ✅ "This decision affects the architecture. Should I document it in MASTER_PLAN.md?"
- ✅ "We've discovered a blocker. Let me update the task spec and TODO.md."
- ✅ "It's been a week since last review. Time to update progress percentages?"
- ✅ "Before merging, let's verify all tests pass."

## Session End Checklist

**Before ending work session:**
- [ ] All code changes committed
- [ ] Documentation updated (TODO.md at minimum)
- [ ] Task status current (in-progress or completed)
- [ ] Activity log entry added
- [ ] Next task identified in TODO.md
- [ ] User knows what's next

**Quality checks:**
- [ ] No TypeScript errors
- [ ] No console warnings
- [ ] Manual testing done
- [ ] Documentation matches reality
- [ ] Progress percentages accurate

## Project-Specific Commands

**Development:**
```bash
npm install           # Install dependencies
npm run compile       # Compile TypeScript
npm run watch         # Watch mode
npm run test          # Run tests
npm run lint          # Run linter
npm run vscode:prepublish  # Package extension
```

**Testing MCP Server:**
```bash
./test-data/scripts/test-mcp-server.sh  # Validate MCP server setup
```

## Important Files to Know

**Configuration:**
- `package.json` - Extension manifest
- `tsconfig.json` - TypeScript configuration
- `.vscode/launch.json` - Debug configuration

**Source:**
- `src/extension.ts` - Extension entry point
- `src/providers/registryTreeProvider.ts` - Tree view
- `src/services/registryService.ts` - API client
- `src/commands/` - Command implementations

**Documentation:**
- `docs/TODO.md` - Current tasks
- `docs/MASTER_PLAN.md` - Project roadmap
- `docs/ai-integration/AI_MCP_INTEGRATION_OPTIONS.md` - AI integration analysis
- `docs/planning/VSCODE_PLUGIN_PLAN.md` - Original vision

## Remember

**"Documentation is not optional. It's part of completing the task."**

**"If documentation is not updated, the task is NOT complete."**

**"Tests must pass before merging to main."**

**"Always use feature branches, never commit directly to main."**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Apicurio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
