## pyplayground

> This document provides a quick reference for contributors working on the **pyplayground**

# Repository Guidelines

This document provides a quick reference for contributors working on the **pyplayground**
project. It covers folder layout, build and test commands, coding style, testing,
and pull‑request expectations.

## Project Structure & Module Organization

- `pyplayground/` – Python source tree (modules, utils, vault helpers).
- `tests/` – unit / integration tests using **pytest**.
- `ansible/` – Ansible playbooks and roles used to provision Vault clusters.
- `docs/`, `examples/`, `templates/` – documentation, example playbooks and
  Jinja templates.
- `scripts/` – helper shell utilities for CI or local dev.

All Python packages expose a public API via `__init__.py`. The root contains
`pyproject.toml` and the runtime dependencies (`requirements.txt`).

## Build, Test, and Development Commands

Before running any Python command, activate the local virtual environment:

```bash
source .venv/bin/activate  # macOS/Linux
# or on Windows: .\venv\Scripts\activate
```

After activation you can use the following commands:

- **Install locally** – `pip install -e .`
- **Run tests** – `pytest` (covers 80% by default). Add flags such as `-v` for verbose.
- **Lint & format** – `flake8`, `black --check`, and `isort --check-only`.
- **Validate Ansible** – `ansible-playbook -i inventory/hosts <playbook>.yml`
  or run the role tests in `ansible/tests`.

## Development Commands

All Python commands must run with the virtual environment activated or by prefixing the executable path.

### Environment Setup

```bash
python -m venv .venv
source .venv/bin/activate
# Install development dependencies (includes production deps)
.venv/bin/pip install -r requirements-dev.txt
```

### Testing

```bash
source .venv/bin/activate
.venv/bin/pytest --cov=pyplayground tests/
```

### Code Quality

```bash
source .venv/bin/activate
.venv/bin/black pyplayground/
.venv/bin/isort pyplayground/
.venv/bin/flake8 pyplayground/
.venv/bin/mypy pyplayground/
```

## Code Organization and Reuse Policy

**MANDATORY CODE REUSE POLICY**: All reusable code MUST be placed in utility libraries. NEVER duplicate code across scripts.

The `pyplayground/utils/` module provides common functionality used across the codebase:

- **Config utilities** (`config_utils.py`): Environment variables, JSON config loading
- **Kubernetes utilities** (`k8s_utils.py`): K8s client, kubeconfig from Vault, node/machine operations
- **Vault utilities** (`vault_utils.py`): Vault client, secret collection, path validation
- **Logging utilities** (`logging_utils.py`): Structured logging setup
- **Migration utilities** (`migration_utils.py`): Secret name normalization, PVC validation
- **Ansible Tower utilities** (`ansible_tower_utils.py`): AWX/Tower client operations

**When writing code:**

1. **BEFORE writing any function**: Check if similar functionality exists in `pyplayground/utils/`
2. **IF functionality is used in 2+ places**: Extract it to the appropriate utils module
3. **IF creating new utility**: Add it to the correct utils module based on its purpose
4. **ALWAYS import from utils**: Use `from pyplayground.utils import function_name`
5. **NEVER copy-paste code**: Refactor to use shared utilities instead

**Python Code Standards**

- **Python version**: 3.9-3.14
- **Line length**: 180 characters maximum
- **CLI frameworks**: Click or Typer for command-line interfaces
- **Type hints**: Required for all functions and classes
- **Docstrings**: Required for all modules, classes, and functions (Google style)
- **Logging**: Use `pyplayground/utils/logging_utils.py` for structured logging

**Required to pass:**

- **black**: Code formatting (enforced via pre-commit)
- **isort**: Import sorting (enforced via pre-commit)
- **flake8**: Linting with docstring checks
- **mypy**: Type checking (strict mode enabled)

All Python code MUST pass black, isort, and flake8 checks before committing.

## Coding Style & Naming Conventions

Python follows *PEP 8* with an enforced line‑length of 88 characters. Use
`black`, `isort`, and `flake8` for formatting. Class names are **PascalCase**;
module and function names use **snake_case**. Constants are UPPER_SNAKE_CASE.
All modules expose a minimal public API – keep imports explicit.

## Markdown Code Block Language Specification

**Rule**: All fenced code blocks MUST have a language identifier specified to comply with MD040/fenced-code-language linting rules.

### Requirements

- Every code block using triple backticks (```) MUST include a language identifier
- If no specific language applies, use `text` as the default language identifier
- Never create code blocks with opening ``` without a language specifier

### Examples

**Correct**:

```python
print("Hello World")
```

```bash
echo "Hello World"
```

```text
This is plain text content
No specific language applies
```

**Incorrect**:

```
This violates MD040
```

### Common Language Identifiers

- Programming: `python`, `bash`, `javascript`, `java`, `yaml`, `json`, `xml`
- Output/Logs: `text`, `console`, `log`
- Documentation: `markdown`, `html`, `css`
- Configuration: `ini`, `toml`, `conf`
- When in doubt: `text`

This rule is clear, actionable, and includes examples of both correct and incorrect usage. It fits well with your existing Ansible documentation standards and will prevent MD040 violations in any markdown files Claude creates for you.

## Documentation Standards

**CRITICAL**: Follow these rules when working with documentation:

- **No emojis or icons** - Documentation must be professional and text-only
- **Ask before creating** - Always ask the user for approval before generating or modifying documentation files
- **No unsolicited documentation** - Never proactively create README files, markdown documentation, or similar without explicit user request
- Main documentation in `docs/` directory
- Update documentation when making code changes

**This applies to all documentation including:**

- README files
- Markdown documentation (*.md)
- Code comments and docstrings (emojis prohibited)
- Commit messages (emojis prohibited)

### Documentation Structure (When Maintained)

**Two Core Documents**:

1. **`docs/progress.md`** - Timeline of what's been done
2. **`docs/debugging.md`** - Issues found and solutions

**progress.md Template**:

```markdown
# Project Progress Tracking

## Phase X: [Phase Name]

### Phase X Task Y: [Task Name]

**Status**: ✅ Complete / 🔄 In Progress / ⏸️ Blocked  
**Date**: YYYY-MM-DD  
**Branch**: feature/branch-name  
**Commit**: [commit-hash]  

**Changes Made**:
- Specific change 1 (with rationale if non-obvious)
- Specific change 2
- Logging added: info/debug/error levels
- Error handling added: try/except/finally

**Tests**:
- Manual: [PASS/FAIL with description]
- Automated: pytest (X/Y passing)
- Validation: [comparison tool / other checks]

**Logging Added/Verified**:
- User actions: logger.info()
- Flow details: logger.debug()
- Exceptions: logger.error(exc_info=True)

**Issues Found**:
[Link to debugging.md entry if any issues discovered]

**Files Modified**:
- path/to/file1.py (+X, -Y lines)
- path/to/file2.py (+A, -B lines)

**Next Steps**: Task Y+1 ([brief description])
```

**debugging.md Template**:

```markdown
# Debugging Log

## Phase X Issues

### Issue: [Brief Description]

**Found During**: Phase X Task Y  
**Date**: YYYY-MM-DD  
**Severity**: CRITICAL / HIGH / MEDIUM / LOW  
**Status**: FIXED / INVESTIGATING / DEFERRED (with reason)

**Symptom**:
[What went wrong - user perspective]

**Root Cause**:
[Why it happened - technical analysis]

**Solution**:
[How it was fixed - implementation details]

**Code Location**:
- File: `path/to/file.py`
- Lines: XXX-YYY
- Function: `functionName()`

**Verification**:
[How we confirmed the fix works]
- Test: [specific test that now passes]
- Manual: [manual verification performed]
- Logs: [relevant log entries]

**Prevention**:
[How to avoid this issue in future]
- Pattern to follow: [code pattern]
- Check to add: [automated check if possible]
- Documentation: [what to document]

**Related Issues**:
- [Link to similar issues if any]
```

**Documentation Principles**:
1. **Update immediately** - Don't defer documentation
2. **Be specific** - "Fixed bug" is not sufficient
3. **Include code** - Show before/after when relevant
4. **Link between docs** - Cross-reference progress.md ↔ debugging.md
5. **No emojis in professional docs** - Text only (except status indicators)

For detailed documentation templates and examples, see `docs/DEVELOPMENT_STANDARDS.md` Section 6.

## Testing Guidelines

The test suite is located under `tests/` and runs with **pytest**. Test files
should be named `test_*.py`. Use the `@pytest.mark.parametrize` decorator for
data‑driven tests. Coverage should hit at least 80 % – run `coverage run -m pytest`
and review `coverage report`.

### Three-Tier Testing Strategy

**Tier 1: Automated Tests** (pytest or equivalent)
- Run after EVERY code change
- Must remain passing (no tolerance for breaking tests)
- Continuous validation required

**Tier 2: Programmatic Validation**
- Use when GUI/integration testing not available
- Methods: syntax validation, pattern verification, round-trip testing, API compliance checking
- Test what CAN be tested without full environment

**Tier 3: Manual Testing**
- Use when full environment available
- Must be structured (not ad-hoc)
- Document each step result and capture logs

**Key Rules**:
- Test at highest available tier - Don't skip testing because ideal environment unavailable
- Never skip Tier 1 - Automated tests always run
- Document test strategy - Explain which tier used and why

**Test Coverage Requirements**:
- Critical paths: 100% (must have tests)
- User-facing features: 90%+ (should have tests)
- Utility functions: 70%+ (nice to have tests)
- Legacy code: Test during modification (add as you touch)

For detailed testing methodology, see `docs/DEVELOPMENT_STANDARDS.md` Section 2.

## Commit & Pull Request Guidelines

### Commit messages

- Follow **Conventional Commits** style: e.g., `feat`, `fix`, `docs`, `refactor`.
  Include a short subject and, if needed, a body with the rationale.

**Standard Format** (all sections required):

```text
Phase X Task Y: [Short description - 50 chars max]

Changes:
- [What changed and WHY - be specific]
- [Added logging: levels used]
- [Added error handling: pattern used]
- [Refactored: what and why]

Testing:
- Manual: [Specific test performed and result]
- Automated: [pytest status - X/Y passing]
- Validation: [comparison tool / other checks]

Logging:
- [What's now logged - be specific about levels]
- [New logger.info(): user actions]
- [New logger.debug(): technical details]
- [New logger.error(): exception handling]

Documentation:
- progress.md updated [what was added]
- debugging.md updated [if applicable]
- Code comments added [where and why]

Files Modified:
- path/to/file1.py (+X, -Y lines): [what changed]
- path/to/file2.py (+A, -B lines): [what changed]

Next: Task Y+1 ([brief description of next task])
```

**Commit Message Rules**:
1. First line: 50 characters max, imperative mood
2. All sections required (even if empty, say "None")
3. Be specific: "Fixed bug" → "Fixed memory leak in dialog cleanup"
4. Reference issues: "Closes #123" if applicable

**Commit Frequency**:
- **One commit per task** - no more, no less
- After task completion
- After all tests passing
- After documentation updated

### Pre-Commit Checklist

**Before EVERY commit**:

```bash
# 1. All tests passing
pytest tests/

# 2. Code quality checks (if applicable)
black pyplayground/
isort pyplayground/
flake8 pyplayground/
mypy pyplayground/

# 3. Documentation updated
git diff docs/progress.md  # Verify entry added
[ -f docs/debugging.md ] && git diff docs/debugging.md  # If issues found

# 4. Commit message prepared
# Use template above

# 5. Review changes
git diff --staged

# 6. Commit
git commit -m "[message]"

# 7. Verify
git log -1 --stat
```

### Pull requests

- Reference an issue number in the title or description.
- Provide clear steps to reproduce changes.
- Attach screenshots for UI/CLI output when relevant.
- Request review from at least one core maintainer.

For detailed git workflow and branch strategy, see `docs/DEVELOPMENT_STANDARDS.md` Section 7.

## Git Commit Messages

**IMPORTANT**: Do NOT add attribution or co-authorship to commit messages.

**CRITICAL**: Never use `git commit --trailer` flags or add any trailers to commit messages.

Commit messages should:

- Follow conventional commit format when appropriate
- Be concise and descriptive
- Focus on the "why" rather than the "what"
- Match the repository's existing commit style
- **NOT include** any branding, attribution, or co-authorship footers
- **NOT use** `--trailer` flags (e.g., `--trailer "Co-authored-by: ..."`)

**Forbidden**:
```bash
# ❌ DO NOT USE
git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>" -m "message"
git commit -m "message" --trailer "Co-authored-by: ..."
```

**Correct**:
```bash
# ✅ USE THIS
git commit -m "message"
```

## System Commands Policy

**CRITICAL**: The `rm` command is strictly prohibited without explicit user permission.

- Never execute `rm` commands without prior approval from the user
- All file deletion operations must be reviewed and authorized by the user first
- When working with temporary files or directories, always ask for confirmation before removal
- If a script needs to delete files, implement an interactive prompt that requires explicit user consent

This policy protects against accidental data loss during development and testing activities.

## Phase & Task Structure

### Phase Definition

**Phase = Major Goal** with clear deliverable

**Requirements**:
- **Duration**: 2-4 weeks typical (flexible)
- **Self-contained**: Can merge/pause at phase boundaries
- **Clear goal**: One sentence description
- **Measurable success**: Specific completion criteria

**Phase Structure**:
```markdown
# Phase X: [Name]

**Goal**: One clear sentence describing what this phase accomplishes

**Duration**: Estimated time
**Priority**: CRITICAL / HIGH / MEDIUM / LOW
**Dependencies**: What must be done first

**Success Criteria**:
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All tests passing

**Tasks**: 6-10 tasks typically
**Deliverables**: What exists at end of phase
```

### Task Definition

**Task = Specific Deliverable** completed in one session

**Requirements**:
- **Duration**: 2-8 hours typical
- **Atomic**: Complete in one sitting
- **Sequential**: Builds on previous tasks
- **Documented**: Creates/updates documentation

**Task Structure** (MANDATORY - all sections required):
- Priority, Effort, Status
- Goal (one clear sentence)
- Prerequisites Checklist
- Sub-tasks
- Logging Requirements
- Testing Requirements
- Documentation Requirements
- Git Commit
- STOP and Report
- Success Criteria

For detailed task template and enforcement, see `docs/DEVELOPMENT_STANDARDS.md` Section 3.

## Debugging Methodology

### Anti-Whack-a-Mole Strategy

**Core Principle**: Fix root causes, not symptoms.

**Rule 1: Fix Bugs in Current Phase**
- ❌ "Low priority, fix later" → Creates technical debt
- ✅ "Fix now, regardless of size" → Prevents accumulation
- ✅ "Document in debugging.md" → Builds knowledge

**Rule 2: Investigation Before Implementation**
- ❌ Jump straight to fixing
- ✅ Thorough investigation task first
- ✅ Document root cause
- ✅ Plan fix approach
- ✅ Then implement

**Rule 3: Use Reference Implementations**
- ❌ Debug blindly when stuck
- ✅ Check reference implementation first
- ✅ Use comparison tools
- ✅ Validate assumptions
- ✅ Copy proven patterns

**Rule 4: Document Everything**
- ❌ "Fixed bug, move on"
- ✅ Document WHY it happened
- ✅ Document HOW to prevent
- ✅ Add to debugging.md
- ✅ Create test to prevent regression

**Standard Debugging Workflow**:
1. Bug discovered → STOP - Don't fix yet
2. Create investigation task → Understand root cause
3. Document in debugging.md → Symptom, root cause, proposed solution
4. Create fix with logging → Follow code pattern template
5. Test fix thoroughly → Automated + manual verification
6. Update documentation → progress.md + debugging.md
7. Git commit → Detailed message
8. STOP for approval → Show evidence, await review

For detailed debugging methodology and workflow, see `docs/DEVELOPMENT_STANDARDS.md` Section 4.

## Reference Documentation

For comprehensive development standards, testing methodology, debugging practices, and detailed templates, see:

- **`docs/DEVELOPMENT_STANDARDS.md`** - Complete development standards and methodology (1147 lines)
- **`AGENT_RULES.md`** - Minimum required rules for all agents (highest precedence)
- **`CLAUDE.md`** - Tool-specific guidance for Claude Code

- Avoid hard‑coding Vault tokens; use environment variables such as
  `VAULT_TOKEN` or files in `~/.vault-token`.
- Keep the `ansible.cfg` minimal – enable `hashiCorp/vault` lookup plugin.

Feel free to reach out on the repo's issue tracker for clarification. Happy coding!

---
> Source: [fischerdr/pyplayground](https://github.com/fischerdr/pyplayground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
