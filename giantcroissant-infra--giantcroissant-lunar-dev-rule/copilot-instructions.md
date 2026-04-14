## giantcroissant-lunar-dev-rule

> **Repository**: GiantCroissant Lunar Dev Rule

# Claude Code Instructions

**Repository**: GiantCroissant Lunar Dev Rule
**Purpose**: Develop and maintain reusable agent rules and pre-commit hooks

---

## Quick Context

This is a **meta-repository** - we develop the rule system that other projects use.

- **What we build**: Agent rules + pre-commit hooks for .NET, Node.js, Python, etc.
- **What we ship**: Templates in `templates/` directory
- **What we install**: Shell script that copies templates to target projects

## Core Principle: Keep It Simple

This repository must be:
- ✅ **Easy to understand** - New contributors should grasp it quickly
- ✅ **Well-documented** - Every decision is explained
- ✅ **Tested thoroughly** - Changes must work before shipping
- ✅ **Backwards compatible** - Don't break existing users

## Directory Structure

```
giantcroissant-lunar-dev-rule/
├── templates/          # SOURCE OF TRUTH
│   ├── base/          # Base rules (installed to all projects)
│   ├── stacks/        # Stack-specific rules (dotnet, nodejs, etc.)
│   ├── hooks/         # Pre-commit configurations
│   └── meta/          # Versioning docs
│
├── cli/               # Installation tooling
│   └── install.sh    # Main installer script
│
├── docs/              # User-facing documentation
├── examples/          # Example configurations
│
├── .agent/            # MINIMAL rules for THIS repo only
└── .github/           # CI/CD and issue templates
```

## Important: Avoid Meta-Confusion

**This repo has TWO sets of rules:**

1. **`templates/`** - Full rule system we ship to users (THE PRODUCT)
2. **`.agent/`** - Minimal rules for developing this repo (NOT THE PRODUCT)

When working on this repo:
- ❌ DON'T edit `templates/` thinking you're configuring this repo
- ❌ DON'T run `./cli/install.sh` on this repo (circular confusion)
- ✅ DO edit `templates/` when improving the rules we ship
- ✅ DO reference `.agent/README.md` for development guidelines

## Common Tasks

### 1. Update Base Rules

**Scenario**: Syncing changes from Winged Bean or adding new rules

**Steps**:
1. Edit `templates/base/20-rules.md`
2. Update version in `templates/base/00-index.md`
3. Update `CHANGELOG.md`
4. Test: `./cli/install.sh --stack dotnet --dry-run` in a clean project
5. Commit changes

**Example**:
```bash
# Edit rules
vim templates/base/20-rules.md

# Update version
vim templates/base/00-index.md
# Change: "Version: 1.1.0" → "Version: 1.2.0"

# Document change
vim CHANGELOG.md
# Add: "## [1.2.0] - 2025-XX-XX"
#      "### Added"
#      "- R-NEW-010: New rule description"
```

### 2. Add New Stack

**Scenario**: Adding Python or Node.js support

**Steps**:
1. Create stack directory: `mkdir -p templates/stacks/{stack-name}`
2. Add stack rules: `templates/stacks/{stack-name}/rules.md`
3. Add hooks: `templates/hooks/{stack-name}/pre-commit-config.yaml`
4. Add adapters (optional): `templates/stacks/{stack-name}/adapters/`
5. Add example: `examples/{stack-name}/README.md`
6. Update main `README.md`
7. Test installation
8. Update `CHANGELOG.md`

**Template for new stack rules.md**:
```markdown
# {Stack Name} Stack Rules

Version: 1.0.0
Last Updated: YYYY-MM-DD

## {Category} Conventions

### R-{STACK}-010: {Rule Title}
- **What**: Description
- **Why**: Rationale
- **How**: Implementation
```

### 3. Modify Installation Script

**Scenario**: Improving `cli/install.sh`

**Steps**:
1. Edit `cli/install.sh`
2. Run shellcheck: `shellcheck cli/install.sh`
3. Test dry-run: `./cli/install.sh --stack dotnet --dry-run`
4. Test actual install in clean project
5. Update docs if behavior changed
6. Commit

**Testing**:
```bash
# Create test project
mkdir -p /tmp/test-project
cd /tmp/test-project
git init

# Test installer
/path/to/giantcroissant-lunar-dev-rule/cli/install.sh --stack dotnet

# Verify
ls -la .agent/
cat .pre-commit-config.yaml
```

### 4. Add Pre-commit Hook

**Scenario**: Adding new validation to templates

**Steps**:
1. Decide: Universal (base) or stack-specific?
2. Edit appropriate config:
   - `templates/hooks/base/pre-commit-config.yaml`
   - `templates/hooks/{stack}/pre-commit-config.yaml`
3. If custom validator needed, add to `templates/hooks/python/validators/`
4. Test locally
5. Document in `docs/quickstart.md`
6. Add example usage

### 5. Update Documentation

**Scenario**: Improving user-facing docs

**Files to update**:
- `README.md` - Main overview and quick start
- `docs/quickstart.md` - Step-by-step tutorial
- `docs/customization.md` - Customization guide
- `CHANGELOG.md` - Version history
- `examples/*/README.md` - Example configurations

**Documentation style**:
- Use clear, concise language
- Include examples
- Provide copy-paste commands
- Explain the "why" not just the "what"

## Development Workflow

### Before Committing

Run all checks:

```bash
# Pre-commit hooks
pre-commit run --all-files

# Shellcheck
shellcheck cli/install.sh

# Test installer (dry run)
./cli/install.sh --stack dotnet --dry-run

# Test installer (actual, in clean project)
cd /tmp/test-project && git init
/path/to/repo/cli/install.sh --stack dotnet
```

### Release Process

1. Update versions:
   - `templates/base/00-index.md` (base version)
   - `templates/stacks/*/rules.md` (stack versions if changed)
   - `CHANGELOG.md` (new version entry)
   - `README.md` (update version references)

2. Test thoroughly:
   ```bash
   # Test all stacks
   ./cli/install.sh --stack dotnet --dry-run
   # (Add more stacks when available)
   ```

3. Commit and tag:
   ```bash
   git commit -m "chore: Release v1.1.0"
   git tag -a v1.1.0 -m "Release v1.1.0

   See CHANGELOG.md for details"
   git push origin main --tags
   ```

4. Create GitHub release with CHANGELOG excerpt

## Security Guidelines

- **Never commit secrets** - Use `.gitignore`
- **Validate all input** - Especially in `install.sh`
- **Use `set -e`** - Fail fast in shell scripts
- **Test edge cases** - Invalid stacks, missing dependencies
- **Review Python code** - Validators run on user machines

## Code Style

### Shell Scripts
- Use `#!/usr/bin/env bash`
- Add `set -e` at top
- Quote variables: `"$VARIABLE"`
- Use functions for reusability
- Add comments for complex logic

### Markdown
- Use GitHub-flavored markdown
- Code blocks with language tags
- Include practical examples
- Keep line length reasonable (no hard limit)

### Python
- Follow PEP 8
- Add docstrings
- Use type hints
- Keep dependencies minimal

## Testing Strategy

### Manual Testing
1. **Dry-run test**: `--dry-run` flag
2. **Clean project test**: Fresh git repo
3. **Pre-commit test**: Run all hooks
4. **Multi-OS test**: macOS and Linux

### Automated Testing (CI/CD)
- GitHub Actions run on push/PR
- Tests: installation, shellcheck, markdown lint, pre-commit
- See `.github/workflows/`

## Common Pitfalls

### ❌ DON'T: Run installer on this repo
```bash
# This creates circular confusion
./cli/install.sh --stack dotnet  # ❌ DON'T DO THIS
```

### ❌ DON'T: Duplicate rules
Don't copy rules from `templates/base/` into `.agent/` here.

### ❌ DON'T: Edit templates thinking you're configuring this repo
`templates/` is what we ship, not our config.

### ✅ DO: Test in clean projects
Always test installation in a separate project.

### ✅ DO: Update CHANGELOG.md
Document all changes for users.

### ✅ DO: Keep backwards compatibility
Avoid breaking changes when possible.

## Reference Materials

- **Development guide**: `.agent/README.md`
- **Extraction notes**: `EXTRACTION_SUMMARY.md`
- **User docs**: `README.md`, `docs/`
- **Examples**: `examples/`
- **Base rules we ship**: `templates/base/20-rules.md`

## Questions?

1. Check `.agent/README.md` for development guidelines
2. Review existing code in `templates/` and `cli/`
3. Look at examples in `examples/`
4. Read `EXTRACTION_SUMMARY.md` for design decisions

---

**Version**: 1.0.0
**Last Updated**: 2025-10-08
**Maintained By**: apprenticegc

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GiantCroissant-Infra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GiantCroissant-Infra)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
