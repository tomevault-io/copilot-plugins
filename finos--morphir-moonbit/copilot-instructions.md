## morphir-moonbit

> This repository includes AI agent skills to enhance development workflows. The skills are compatible with multiple AI coding assistants.

# AI Agent Configuration

This repository includes AI agent skills to enhance development workflows. The skills are compatible with multiple AI coding assistants.

## Supported Agents

- **Claude** (Anthropic)
- **GitHub Copilot**
- **OpenAI Codex**
- **Cursor**
- **Windsurf**
- **Google Antigravity**

## Available Skills

### Code Reviewer

**Location**: `.github/skills/code-reviewer/`

**Purpose**: Comprehensive code review for pull requests and code changes in the Morphir Moonbit monorepo.

**Usage**: Reference this skill when you need to review code changes before merging.

**Key Features**:
- Checks Moonbit code quality and formatting
- Validates package structure and dependencies
- Ensures test coverage and quality
- Verifies build configuration
- Reviews documentation updates
- Identifies security issues
- Validates mise tasks and CI workflows

## How to Use

### For Developers

When working with an AI coding assistant, you can reference the skills:

```
Please review my code changes using the code-reviewer skill.
```

### For AI Agents

Skills are located in `.github/skills/` and follow the Agent Skills specification (agentskills.io).

Each skill includes:
- `SKILL.md`: Skill definition with metadata and instructions
- Additional resources as needed

## Agent-Specific Instructions

Different AI agents may load skills differently. Check your agent's documentation for specific instructions on loading skills from this repository.

### Agent Instruction Files

For agents that support configuration files in specific locations, symbolic links are provided:

- `.github/copilot/` - GitHub Copilot
- `.cursorrules` - Cursor
- `.windsurfrules` - Windsurf  
- `.claude/` - Claude
- `.codex/` - OpenAI Codex
- `.antigravity/` - Google Antigravity

These files are symbolic links pointing to the shared skill definitions to avoid duplication.

## Adding New Skills

To add a new skill:

1. Create a new directory in `.github/skills/<skill-name>/`
2. Add a `SKILL.md` file following the Agent Skills specification
3. Update this AGENTS.md file to document the new skill
4. Create symbolic links for agent-specific locations if needed

## Pre-Push Requirements

**⚠️ IMPORTANT**: All lint, format, and validation checks **MUST** pass before pushing to the repository.

### Required Checks

Before any `git push`, the following checks must pass:

1. **Linting** (`mise run lint`)
   - YAML file validation
   - Moonbit code style checks

2. **Format Check** (`mise run lint:moonbit`)
   - Moonbit code formatting verification
   - Ensures code is formatted with `moon fmt`

3. **Validation** (`mise run validate`)
   - Package structure validation
   - Configuration file verification

### Automated Enforcement

Git hooks are **automatically installed** when you enter the directory (via mise hooks). You can also manually run:

```bash
# Manually trigger hook setup (idempotent)
mise run setup:hooks
```

This installs a pre-push git hook that:
- Runs all required checks before allowing a push
- Prevents pushing code that fails validation
- Provides clear error messages if checks fail
- Can be bypassed with `git push --no-verify` (not recommended)

### Manual Verification

You can run all checks manually before pushing:

```bash
mise run check
```

Or run individual checks:

```bash
mise run lint           # Run linting
mise run format         # Auto-format code
mise run validate       # Run validation
```

### CI/CD Integration

The same checks run in CI/CD pipelines. Commits that fail these checks will:
- Block PR merges
- Fail CI workflows
- Require fixes before merge approval

**Always run checks locally before pushing to avoid CI failures.**

## Contributing

When adding or modifying skills:

- Follow the Agent Skills specification (agentskills.io)
- Test with multiple agents when possible
- Update documentation
- Keep skills focused and modular
- **Ensure all pre-push checks pass**

## References

- [Agent Skills Specification](https://agentskills.io/specification)
- [Project Contributing Guide](../CONTRIBUTING_DEV.md)
- [Project Architecture](../docs/ARCHITECTURE.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
