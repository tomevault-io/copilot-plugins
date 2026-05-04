## securevibes

> This file defines coding standards, workflows, and conventions for AI assistants (Droid/Claude) working on SecureVibes. Factory automatically reads this file to ensure consistent, high-quality results.

# SecureVibes Agent Guidelines

This file defines coding standards, workflows, and conventions for AI assistants (Droid/Claude) working on SecureVibes. Factory automatically reads this file to ensure consistent, high-quality results.

---

## Git Workflow

### Staging and Committing

**STOP POINT: After `git add` + commit message suggestion**

When making changes, Droid should:
1. ✅ Make changes (create, edit, delete files)
2. ✅ Stage changes with `git add <files>`
3. ✅ Generate a commit message following repository style (check `git log --oneline -10`)
4. ✅ Display the full command: `git commit -m "suggested message"`
5. 🛑 **STOP HERE** - Wait for manual execution

**I will then:**
- Review staged files with `git status` (optional)
- Review changes with `git diff --cached` (optional)
- Execute or modify the suggested commit command
- Push manually when ready

**Droid should NEVER:**
- ❌ Run `git commit` (unless explicitly requested)
- ❌ Run `git push` (unless explicitly requested)
- ❌ Commit without showing what's staged first

**Example:**
```bash
# Droid does:
git add packages/core/securevibes/scanner.py tests/test_scanner.py

# Then outputs:
"✅ Files staged: scanner.py, test_scanner.py

Suggested commit command:
git commit -m "feat: Add file type filtering to scanner"

Review with 'git status' or 'git diff --cached' if needed."
```

---

## Code Style

### Python Standards

- **Follow PEP 8** for all Python code
- **Type hints**: Use type annotations for function signatures
- **Docstrings**: Use for all public functions/classes (Google style)
- **Line length**: 100 characters (configured in pyproject.toml)
- **Imports**: Group in standard order (stdlib, third-party, local)

### Formatting Tools

- **Black**: Configured in `pyproject.toml` (line-length = 100)
- **Ruff**: Linter configured in `pyproject.toml`

### Code Patterns

**Prefer:**
- ✅ Early returns over nested conditionals
- ✅ Composition over inheritance
- ✅ Explicit over implicit
- ✅ Small, focused functions (under 30 lines)
- ✅ Descriptive variable names

**Avoid:**
- ❌ Broad exception catching without re-raising
- ❌ Mutable default arguments
- ❌ Global state
- ❌ Magic numbers (use constants)

---

## Testing

### Test Requirements

**Before completing any feature:**
1. ✅ Run relevant tests: `pytest packages/core/tests/`
2. ✅ Check for import errors
3. ✅ Verify documentation examples still work
4. ✅ Ensure all tests pass

**When changing code, always update together:**
- ✅ Code implementation
- ✅ Tests for the code
- ✅ Documentation (docstrings, README, etc.)
- ✅ Remove docs for deleted features

**Never:**
- ❌ Update code without updating tests
- ❌ Delete features without removing docs
- ❌ Batch documentation updates separately

### Test Commands

```bash
# Run all tests
pytest packages/core/tests/

# Run with coverage
pytest packages/core/tests/ --cov=securevibes --cov-report=term-missing

# Run specific test file
pytest packages/core/tests/test_scanner.py

# Run with verbose output
pytest packages/core/tests/ -v
```

### Test Structure

Follow the pattern in `packages/core/tests/test_scanner.py`:
- Use descriptive test names: `test_<function>_<scenario>_<expected_result>`
- Use fixtures for common setup
- Keep tests focused and independent
- Use `pytest-asyncio` for async tests

---

## Architecture

### Multi-Agent Pattern

SecureVibes uses a specialized agent architecture:
- **Assessment Agent**: Maps codebase architecture
- **Threat Modeling Agent**: STRIDE threat analysis
- **Code Review Agent**: Security vulnerability detection
- **Report Generator**: Compiles results

**Follow existing patterns in:**
- `packages/core/securevibes/agents/definitions.py` - Agent definitions
- `packages/core/securevibes/scanner/scanner.py` - Scanner orchestration

### Package Structure

```
packages/core/securevibes/
├── agents/          # Agent definitions and prompts
├── cli/             # CLI interface
├── config.py        # Configuration management
├── models/          # Data models (Issue, Result)
├── prompts/         # Agent prompt templates
├── reporters/       # Output formatters (JSON, Markdown)
└── scanner/         # Core scanning logic
```

**When adding new features:**
- Scanners go in `scanner/`
- Reporters go in `reporters/` with interface matching existing ones
- Models go in `models/` with clear type definitions
- Agent prompts go in `prompts/` as `.txt` files

---

## Common Commands

### Development

```bash
# Run tests
pytest packages/core/tests/

# Format code
black packages/core/securevibes packages/core/tests

# Lint code
ruff check packages/core/securevibes packages/core/tests

# Install in development mode
pip install -e packages/core

# Install with dev dependencies
pip install -e "packages/core[dev]"
```

### Before Declaring "Done"

Run this verification sequence:
```bash
# 1. Format
black packages/core/securevibes packages/core/tests

# 2. Lint
ruff check packages/core/securevibes packages/core/tests

# 3. Test
pytest packages/core/tests/

# 4. Check imports
python -c "import securevibes; print('OK')"
```

---

## Documentation

### Multiple Locations

This project has documentation in multiple locations:
- **Root `README.md`**: Project overview, quick start, features
- **`packages/core/README.md`**: Core package documentation
- **`docs/`**: Detailed guides and maintenance procedures
  - `docs/MAINTENANCE.md`: Full maintenance guide

**When making changes:**
- ✅ Update ALL relevant locations
- ✅ Keep examples consistent across all docs
- ✅ Remove references to deleted features immediately
- ✅ Verify all code examples actually run

### Documentation Standards

- **Code examples**: Always test that they work
- **CLI examples**: Use actual commands that can be copy-pasted
- **API changes**: Update both root and package READMEs together
- **Links**: Use relative paths for internal documentation

---

## Code Examples

### Scanner Pattern

Follow `packages/core/securevibes/scanner/scanner.py`:
- Async/await for agent interactions
- Progress tracking with rich console
- Error handling with specific exception types
- Cost tracking for API calls

### Reporter Pattern

Follow `packages/core/securevibes/reporters/json_reporter.py`:
- Implement base reporter interface
- Clean data serialization
- Handle missing/optional fields gracefully

### Agent Definition Pattern

Follow `packages/core/securevibes/agents/definitions.py`:
- Clear agent roles and responsibilities
- Structured prompt templates
- Type-safe agent configurations

### Test Pattern

Follow `packages/core/tests/test_scanner.py`:
- Use fixtures for setup
- Mock external dependencies
- Test both success and error cases
- Async test patterns with pytest-asyncio

---

## Maintenance

### Regular Checks

Periodically suggest (don't do automatically):
- Dead code detection
- Test coverage reports (`pytest --cov`)
- Documentation audits
- Dependency updates

**Detailed procedures in:** `docs/MAINTENANCE.md`

### Security Checks

Before committing:
- ✅ No hardcoded secrets, API keys, or passwords
- ✅ No sensitive data in logs
- ✅ No `.env` files committed
- ✅ Review `git diff --cached` for sensitive information

---

## Mistakes to Avoid

### Code Quality

- ❌ Don't use `any` type annotations in Python
- ❌ Don't catch exceptions without logging or re-raising
- ❌ Don't leave TODO/FIXME comments without GitHub issues
- ❌ Don't hardcode file paths (use Path objects)

### Git

- ❌ Never commit .env files
- ❌ Never force push to main branch
- ❌ Never commit untested code
- ❌ Never batch unrelated changes in one commit

### Testing

- ❌ Don't skip tests for "small" changes
- ❌ Don't commit with failing tests
- ❌ Don't write tests that depend on external services without mocking
- ❌ Don't ignore test warnings

---

## Communication Style

When presenting work:
- ✅ Be concise (1-4 sentences summaries)
- ✅ Show what changed, not verbose explanations
- ✅ List files modified/created/deleted
- ✅ Indicate when stopped (after git add, before commit)

---

**Last Updated:** 2025-10-11  
**Owner:** @anshumanbh  
**Project:** SecureVibes - AI-Native Security Scanner

---
> Source: [anshumanbh/securevibes](https://github.com/anshumanbh/securevibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
