## arzakops

> This document provides comprehensive guidance for AI assistants working with the ArzakOps codebase. It outlines the project structure, development workflows, and key conventions to ensure consistent and effective collaboration.

# CLAUDE.md - AI Assistant Guide for ArzakOps

This document provides comprehensive guidance for AI assistants working with the ArzakOps codebase. It outlines the project structure, development workflows, and key conventions to ensure consistent and effective collaboration.

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Codebase Structure](#codebase-structure)
3. [Development Workflows](#development-workflows)
4. [Key Conventions for AI Assistants](#key-conventions-for-ai-assistants)
5. [Code Quality Standards](#code-quality-standards)
6. [Git Workflow](#git-workflow)
7. [Common Tasks](#common-tasks)
8. [Testing Guidelines](#testing-guidelines)
9. [Documentation Standards](#documentation-standards)

---

## Repository Overview

**Project Name:** ArzakOps
**Purpose:** [To be defined as project develops]
**Tech Stack:** [To be defined based on initial implementation]

### Project Goals
- Maintain high code quality and maintainability
- Follow consistent coding conventions
- Ensure comprehensive testing coverage
- Document all major components and workflows

---

## Codebase Structure

The repository structure will follow a modular organization pattern:

```
ArzakOps/
├── src/                    # Source code
│   ├── components/        # Reusable components
│   ├── services/          # Business logic and services
│   ├── utils/             # Utility functions
│   ├── config/            # Configuration files
│   └── types/             # Type definitions
├── tests/                 # Test files
├── docs/                  # Documentation
├── scripts/               # Build and utility scripts
└── CLAUDE.md             # This file
```

**Note:** This structure will be updated as the project evolves.

---

## Development Workflows

### Branch Strategy

- **Main Branch:** `main` or `master` - Production-ready code
- **Feature Branches:** `claude/feature-description-sessionid` - For new features
- **Bug Fix Branches:** `claude/fix-description-sessionid` - For bug fixes
- **Development Branch:** Branch names must start with `claude/` and end with session ID

### Development Process

1. **Always work on feature branches** - Never commit directly to main
2. **Branch naming:** Use descriptive names that indicate the purpose
3. **Commit frequently** with clear, descriptive messages
4. **Push changes** when a logical unit of work is complete
5. **Create pull requests** for code review before merging

---

## Key Conventions for AI Assistants

### General Principles

1. **Read Before Writing**
   - ALWAYS read existing files before modifying them
   - Understand the current implementation before suggesting changes
   - Never propose changes to code you haven't examined

2. **Minimal Changes**
   - Only modify what's necessary to fulfill the request
   - Avoid refactoring unless explicitly asked
   - Don't add features beyond what's requested
   - Keep solutions simple and focused

3. **Code Quality**
   - Follow existing code style and patterns
   - Maintain consistency with the current codebase
   - Write clean, readable, self-documenting code
   - Add comments only where logic isn't self-evident

4. **Security Awareness**
   - Never introduce security vulnerabilities
   - Watch for: command injection, XSS, SQL injection, path traversal
   - Validate input at system boundaries
   - Don't over-engineer error handling for internal code

5. **Backwards Compatibility**
   - Avoid compatibility hacks unless necessary
   - If code is unused, delete it completely
   - Don't rename variables to `_var` or add `// removed` comments
   - Clean removal is better than deprecated code

### File Operations

1. **Prefer Editing Over Creating**
   - ALWAYS prefer editing existing files
   - Only create new files when absolutely necessary
   - Don't create documentation files unless explicitly requested

2. **Use Appropriate Tools**
   - Use `Read` for reading files, not `cat`
   - Use `Edit` for modifications, not `sed/awk`
   - Use `Write` only for new files
   - Use `Grep` for searching, not bash grep

3. **Code References**
   - When referencing code, use format: `file_path:line_number`
   - Example: "The error handler is in src/services/api.ts:142"

### Communication Style

1. **Concise and Direct**
   - Keep responses short and focused
   - Avoid unnecessary explanations
   - Don't use emojis unless requested
   - Output text directly, not through bash echo

2. **Professional Tone**
   - Focus on technical accuracy
   - Provide objective guidance
   - Respectfully correct when necessary
   - Avoid excessive praise or validation

### Task Management

1. **Use TodoWrite for Complex Tasks**
   - Break down multi-step tasks
   - Track progress transparently
   - Mark tasks complete immediately when done
   - Keep ONE task in_progress at a time

2. **When to Use Todos**
   - Complex multi-step tasks (3+ steps)
   - Non-trivial implementations
   - User provides multiple tasks
   - After receiving new instructions

3. **When NOT to Use Todos**
   - Single straightforward tasks
   - Trivial operations
   - Simple file reads
   - Purely informational requests

---

## Code Quality Standards

### Code Style

- **Consistency:** Match existing code style in the file/module
- **Naming:** Use descriptive, self-explanatory names
- **Functions:** Keep functions focused on a single responsibility
- **Comments:** Only add where logic isn't obvious
- **Formatting:** Follow language-specific conventions

### Error Handling

- **Validate at boundaries:** User input, external APIs, file I/O
- **Trust internal code:** Don't validate what frameworks guarantee
- **Don't catch what you can't handle:** Let errors bubble up appropriately
- **Provide context:** Include relevant information in error messages

### Performance

- **Don't optimize prematurely:** Focus on clarity first
- **Avoid unnecessary abstractions:** Three similar lines > premature abstraction
- **One-time operations:** Don't create utilities for single-use code
- **Design for now:** Not hypothetical future requirements

---

## Git Workflow

### Committing Changes

**Only create commits when explicitly requested by the user.**

#### Pre-Commit Checklist

1. Run `git status` to see all changes (NEVER use `-uall` flag)
2. Run `git diff` to review changes
3. Check `git log` to match commit message style
4. Stage relevant files with `git add`
5. Commit with descriptive message
6. Run `git status` after commit to verify

#### Commit Message Format

```bash
git commit -m "$(cat <<'EOF'
<type>: <brief description>

<optional detailed explanation>
EOF
)"
```

**Commit Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `docs`: Documentation changes
- `test`: Test additions/changes
- `chore`: Maintenance tasks

#### Git Safety Rules

- NEVER update git config
- NEVER run destructive commands without explicit permission
- NEVER skip hooks (--no-verify, --no-gpg-sign)
- NEVER force push to main/master
- NEVER commit files with secrets (.env, credentials.json)
- NEVER use interactive git commands (-i flag)

#### Amending Commits

Only use `git commit --amend` when ALL conditions are met:
1. User explicitly requested amend, OR pre-commit hook auto-modified files
2. HEAD commit was created in current conversation
3. Commit has NOT been pushed to remote

**CRITICAL:** If commit FAILED or was REJECTED, NEVER amend - create a NEW commit

### Pushing Changes

```bash
git push -u origin <branch-name>
```

**Important:**
- Branch must start with `claude/` and end with session ID
- Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s) on network errors
- Example: `git push -u origin claude/feature-auth-abc123`

### Pull Requests

When creating PRs:

1. Check branch status and diff from base branch
2. Review ALL commits (not just the latest)
3. Use `git log` and `git diff [base-branch]...HEAD`
4. Create PR with comprehensive summary:

```bash
gh pr create --title "PR Title" --body "$(cat <<'EOF'
## Summary
- Bullet point 1
- Bullet point 2

## Test Plan
- [ ] Test case 1
- [ ] Test case 2
EOF
)"
```

---

## Common Tasks

### Setting Up Development Environment

```bash
# Clone repository
git clone <repository-url>
cd ArzakOps

# Create feature branch
git checkout -b claude/feature-name-sessionid

# Install dependencies (once defined)
# [To be added based on tech stack]
```

### Running Tests

```bash
# Run all tests
# [To be added based on testing framework]

# Run specific test
# [To be added based on testing framework]
```

### Building the Project

```bash
# Development build
# [To be added based on build system]

# Production build
# [To be added based on build system]
```

### Code Analysis

```bash
# Lint code
# [To be added based on linting tools]

# Type checking
# [To be added based on type system]

# Format code
# [To be added based on formatter]
```

---

## Testing Guidelines

### Testing Principles

1. **Write tests for new features** - Unless explicitly told not to
2. **Maintain existing tests** - Update tests when changing functionality
3. **Test at appropriate levels** - Unit, integration, and E2E as needed
4. **Keep tests simple** - Tests should be easy to understand and maintain

### Test Organization

```
tests/
├── unit/              # Unit tests
├── integration/       # Integration tests
└── e2e/              # End-to-end tests
```

### Test Naming

- Test files: `[module-name].test.[ext]`
- Test cases: Descriptive names that explain what's being tested
- Use clear arrange-act-assert structure

---

## Documentation Standards

### Code Documentation

1. **Self-Documenting Code First**
   - Use clear variable and function names
   - Keep functions focused and simple
   - Structure code logically

2. **Comments When Needed**
   - Explain WHY, not WHAT
   - Document complex algorithms
   - Note important constraints or assumptions
   - Reference tickets/issues for context

3. **Type Annotations**
   - Use TypeScript or equivalent for type safety
   - Document complex type structures
   - Maintain type definitions in dedicated files

### Project Documentation

- **README.md:** Project overview, setup, and usage
- **CLAUDE.md:** This file - AI assistant guide
- **API Documentation:** Document all public APIs
- **Architecture Docs:** Explain system design and decisions

### Documentation Updates

- Update documentation when changing functionality
- Keep examples current with the codebase
- Remove outdated information
- Version documentation with code

---

## Best Practices Summary

### DO:
✅ Read files before modifying them
✅ Keep changes minimal and focused
✅ Follow existing code patterns
✅ Write clear commit messages
✅ Use appropriate tools for each task
✅ Test changes before committing
✅ Ask for clarification when uncertain
✅ Work on feature branches
✅ Push regularly to remote

### DON'T:
❌ Modify code you haven't read
❌ Add features not requested
❌ Refactor without being asked
❌ Create files unnecessarily
❌ Commit directly to main
❌ Skip security considerations
❌ Over-engineer solutions
❌ Add backwards-compatibility hacks
❌ Commit files with secrets
❌ Force push without permission

---

## Getting Help

If you're unsure about:
- Project conventions: Check existing code for patterns
- Task requirements: Ask the user for clarification
- Technical decisions: Research and propose options
- Breaking changes: Discuss impact before implementing

---

## Changelog

### 2026-01-22
- Initial creation of CLAUDE.md
- Established base conventions and workflows
- Set up documentation structure

---

**Last Updated:** 2026-01-22
**Maintained by:** AI Assistants working with ArzakOps
**Review Frequency:** Update when significant project changes occur

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArzakOps) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
