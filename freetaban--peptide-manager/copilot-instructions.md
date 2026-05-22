## peptide-manager

> 1. **ONE TASK AT A TIME**: Focus on a single, atomic task from the TODO list. Never work on multiple tasks simultaneously.

# Cursor Rules - Peptide Management System Refactoring

## 🎯 Core Principles

1. **ONE TASK AT A TIME**: Focus on a single, atomic task from the TODO list. Never work on multiple tasks simultaneously.

2. **THINK BEFORE CODE**: Before writing ANY code:
   - Analyze the current state thoroughly
   - Identify all affected components
   - Consider edge cases and failure modes
   - Evaluate alternative approaches
   - Document the decision rationale

3. **NO HALLUCINATIONS**: 
   - Only use code that exists in the codebase
   - Verify file paths, function names, and APIs before referencing
   - If uncertain, search the codebase first
   - Never assume implementation details

4. **INCREMENTAL CHANGES**:
   - Make small, testable changes
   - Ensure each change compiles/runs before proceeding
   - Maintain backward compatibility during migration
   - Keep the system functional at all times

5. **VALIDATION REQUIRED**:
   - Check for linter errors after every edit
   - Verify imports and dependencies
   - Test affected functionality when possible
   - Document breaking changes

## 📋 Task Execution Workflow

### Before Starting a Task:
1. Read the TODO item carefully
2. Search codebase for all related code
3. Identify dependencies and affected modules
4. Check for existing tests
5. Review similar implementations (if any)
6. Propose approach and get confirmation (if major change)

### During Implementation:
1. Make one logical change at a time
2. Update related files together (models + repository + tests)
3. Run linter after each file edit
4. Update documentation if API changes
5. Maintain type hints and docstrings

### After Completion:
1. Verify no linter errors
2. Check that related tests still pass
3. Update TODO status
4. Document any new patterns or conventions

## 🔍 Code Analysis Requirements

### When Reading Code:
- Read the ENTIRE file, not just snippets
- Understand the context and purpose
- Identify patterns and conventions
- Note dependencies and imports
- Check for error handling

### When Searching:
- Use semantic search for concepts
- Use grep for exact symbols/names
- Check multiple files for related code
- Verify file paths are correct
- Look for similar patterns elsewhere

### When Proposing Changes:
- Show current implementation first
- Explain why change is needed
- List all affected files
- Consider migration path
- Propose backward-compatible solution

## 🚫 Anti-Patterns to Avoid

1. **Don't assume**: Always verify before referencing
2. **Don't copy-paste blindly**: Understand what you're copying
3. **Don't break existing APIs**: Maintain compatibility during migration
4. **Don't skip validation**: Always check linter and tests
5. **Don't mix concerns**: Keep database, business logic, and UI separate
6. **Don't create circular dependencies**: Watch import chains
7. **Don't ignore error handling**: Consider failure modes

## 📝 Documentation Standards

- Update docstrings when changing function signatures
- Document breaking changes in CHANGELOG
- Keep architecture decisions in ARCHITECTURE.md
- Update README if user-facing behavior changes
- Comment complex logic, not obvious code

## 🔄 Migration Strategy

### Database Migrations:
- Always create Alembic revisions for schema changes
- Test migrations on dev database first
- Never modify existing migrations (create new ones)
- Document data migration steps separately
- Verify foreign key constraints

### Code Migrations:
- Keep old code until new code is fully tested
- Use adapter pattern for backward compatibility
- Migrate one module at a time
- Update all call sites before removing old code
- Write tests for new implementation first

## 🧪 Testing Requirements

- Write tests for new repository methods
- Test edge cases (None, empty, invalid input)
- Test error conditions
- Verify backward compatibility
- Run existing tests before and after changes

## 🎨 UI/UX Refactoring Rules

- Separate presentation from business logic
- Create reusable components
- Use service layer for data operations
- Implement proper error handling and user feedback
- Maintain consistent patterns across UI
- Never block UI with synchronous operations

## 🔐 Safety Checks

Before any destructive operation:
- Verify database path (not production unless explicit)
- Check for active connections
- Ensure backups exist
- Confirm user intent for destructive changes

## 📊 Progress Tracking

- Mark TODO as "in_progress" when starting
- Mark as "completed" only when fully done and tested
- Add notes about blockers or decisions made
- Update related documentation

---
> Source: [Freetaban/peptide-manager](https://github.com/Freetaban/peptide-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
