## yourbench-v1-standalone

> - **task-list.md is the SINGLE source of truth** for all **planned work and features**

# Task Management & Development Workflow

## CRITICAL WORKFLOW RULES (MANDATORY)

### 1. TASK LIST AS SOURCE OF TRUTH
- **task-list.md is the SINGLE source of truth** for all **planned work and features**
- **NEVER work on tasks not in the task list** - add them first if needed
- **This does not apply to bug fixes or troubleshooting**
- **ALWAYS reference task numbers** when working (e.g., "Task 2.2: Add autocomplete")
- **Use exact task naming** from the list in commits and documentation
- **At the completion of each task review all project documentation and update as needed**

### 2. TASK STATUS MANAGEMENT
```markdown
- [x] **Completed** - Task is done, tested, and committed
- [i] **In Progress** - Task is currently being worked on
- [ ] **Todo** - Task needs to be started
- [b] **Blocked** - Task is waiting on dependencies
```

**Status Update Rules:**
- Change to [i] when starting work
- Change to [x] ONLY after all tests pass and code is committed
- Document blocking reasons for [b] status
- Include test results in completion notes
- **Task list updates MUST be included in the same commit as the feature implementation**

### 3. ATOMIC TASK COMPLETION PROCESS (MANDATORY)

#### Step 0: Branching Strategy Enforcement
- **ALWAYS check current branch** before starting any task
- **NEVER work on main branch** - immediately create feature branch if on main
- **Use proper branching workflow**: `pnpm feature "task description"`
- **Follow emergency recovery** if found working on main (see development-standards.mdc)
- **Verify main branch health** - ensure main has all tests passing before creating feature branch

#### Step 1: Task Selection
- **ALWAYS ask user before starting a new task**
- Reference specific task number and description
- Confirm task priority and dependencies
- Example: "Ready to work on Task 2.2: Add autocomplete functionality with @ symbol?"

#### Step 2: Implementation
- **Follow TDD approach** - write tests first (see testing-standards.mdc)
- **Make atomic changes** - small, focused commits
- **Document all code** with JSDoc comments
- **Test continuously** during development
- **For ANY bugs encountered**: Write test that reproduces the bug BEFORE fixing it (see testing-standards.mdc for bug-driven test development)

#### Step 3: Testing & Main Branch Protection
- **MUST run full test suite** after task completion
- **ALL tests must pass** before marking task complete
- **ALL tests must pass** before merging to main branch
- **Fix any broken tests** immediately using `./scripts/fix-tests.sh`
- **Add new tests** for new functionality
- **Document test results** in task completion

#### Step 4: Task Completion
- **Run final test suite**: `pnpm test`
- **Update task-list.md** status to [x] in the same commit as the feature
- **Get user approval for commit message** before committing - **MANDATORY**
- **Ask user before proceeding** to next task

### 4. COMMIT MESSAGE STANDARDS
```
feat: [Task X.Y] Brief description

✨ New Features:
- Specific feature 1
- Specific feature 2

🧪 Testing:
- X tests added/updated
- All tests passing ✅
- Test coverage: X%

📚 Documentation:
- JSDoc added for all functions
- Updated relevant documentation

🌿 Branch Info:
- Branch: feature/task-description
- Ready for self-review: `pnpm review`

🎯 Task Status:
- ✅ Task X.Y: [Task Name] - COMPLETED
```

### 5. USER CONFIRMATION REQUIREMENTS

#### Before Starting New Task:
```
🎯 Next Task: [Task X.Y] - [Task Name]
📋 Description: [Brief description]
🔗 Dependencies: [Any dependencies]
❓ Ready to proceed? (Yes/No)
```

#### Before Committing (MANDATORY - CRITICAL):
```
📝 Proposed Commit Message:
[Commit message here]

🎯 Changes to be committed:
- [List of changes]
- [Task list update included]

❓ Approve this commit? (Yes/No/Edit)
```

**CRITICAL RULES:**
- **NEVER commit without user approval** - this is a MANDATORY requirement
- **ALWAYS show the exact commit message** before committing
- **Wait for explicit "Yes" response** before executing git commit
- **If user says "Edit"**: Ask for specific changes and show updated message
- **If user says "No"**: Do not commit, ask for guidance
- **This applies to ALL commits** - no exceptions, no shortcuts
- **Documentation updates, bug fixes, and any git commits** require this confirmation

#### After Task Completion:
```
✅ Task Completed: [Task X.Y] - [Task Name]
🧪 Tests: [X/Y passing] ([success rate]%)
📝 Commit: [commit hash]
🎯 Next suggested task: [Task X.Y] - [Task Name]
❓ Proceed to next task? (Yes/No)
```

### 6. TASK LIST MAINTENANCE

#### Weekly Reviews:
- Review completed tasks and update priorities
- Add new requirements as they emerge
- Update time estimates based on actual progress
- Reorganize tasks based on dependencies

#### Task Updates:
- Add completion dates to finished tasks
- Include test results and commit references
- Document any issues or blockers encountered
- Update dependent tasks when prerequisites complete
- **Task list updates MUST be included in feature commits, not separate commits**

#### Scope Clarification:
- **task-list.md is for planned work and features only**
- **Bug fixes, cleanup, and tech debt do NOT need task list entries**
- **Focus on user-facing features and major architectural changes**

### 7. COLLABORATION PATTERNS

#### Starting Work Session:
1. Review current task list status
2. Identify next priority task
3. Confirm with user before proceeding
4. Update task status to [i] In Progress
5. **Create dedicated branch for task work**

#### During Development:
1. **ALWAYS start with a test** before writing any implementation code
2. Write minimal code to pass the test
3. Test frequently and fix issues immediately
4. Document all changes with JSDoc
5. **For UI/UX changes, get user visual approval before committing**

#### Ending Work Session:
1. Run full test suite
2. **Get user approval for commit message**
3. **Update task list in same commit as feature**
4. Commit changes with user approval
5. Ask user about next priorities

### 8. EMERGENCY PROCEDURES

#### If Tests Fail:
1. **STOP all other work immediately**
2. **IDENTIFY failing tests** and root cause
3. **FIX issues** before proceeding using `./scripts/fix-tests.sh`
4. **VERIFY fix** with test re-run
5. **DOCUMENT fix** in commit message
6. **NEVER merge to main** until all tests are passing

#### If Main Branch Has Failing Tests:
1. **STOP all feature development immediately**
2. **IDENTIFY the commit that broke tests**
3. **REVERT the problematic commit** or fix the failing tests
4. **VERIFY main branch is clean** - all tests must pass
5. **RESUME feature development** only after main is stable

#### If Task Blocked:
1. **UPDATE task status** to [b] Blocked
2. **DOCUMENT blocking reason** clearly
3. **IDENTIFY alternative tasks** that can proceed
4. **ASK USER** for guidance on priorities

#### If Requirements Change:
1. **PAUSE current work** if needed
2. **UPDATE task-list.md** with new requirements
3. **REASSESS priorities** with user
4. **ADJUST current task** scope if necessary

### 9. SUCCESS METRICS

#### Task Completion Quality:
- ✅ All tests passing
- ✅ Full documentation added
- ✅ Atomic commits with clear messages
- ✅ Task list updated accurately
- ✅ User confirmation received

#### Development Velocity:
- Track tasks completed per session
- Measure test coverage improvements
- Monitor code quality metrics
- Document lessons learned

---

## QUICK REFERENCE

### Before Each Task:
- [ ] Confirm task with user
- [ ] Update status to [i]
- [ ] Verify main branch has all tests passing
- [ ] Create dedicated branch for task work
- [ ] Write tests first (TDD)

### During Each Task:
- [ ] Start with test before implementation
- [ ] Write minimal code to pass test
- [ ] Test continuously
- [ ] Document with JSDoc
- [ ] For UI/UX: get visual approval before committing

### After Each Task:
- [ ] Run full test suite
- [ ] All tests passing ✅
- [ ] **Get user approval for commit message** - **MANDATORY**
- [ ] Update task status to [x] in same commit
- [ ] Commit with user approval
- [ ] Verify all tests still pass before merging to main
- [ ] Ask user about next task

### Never Do:
- ❌ Work without user confirmation
- ❌ Skip testing requirements
- ❌ Make large, complex changes
- ❌ Commit without documentation
- ❌ **Commit without user approval** - **CRITICAL RULE**
- ❌ Update task list in separate commit
- ❌ Work on main branch
- ❌ Merge to main when tests are failing
- ❌ Allow main branch to have broken tests

---

*This workflow ensures high-quality, collaborative development with clear progress tracking and user involvement at every step.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TaylorHuston)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TaylorHuston)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
