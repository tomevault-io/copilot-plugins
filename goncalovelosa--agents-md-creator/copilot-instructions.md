## agents-md-creator

> |


# AGENTS.md Creator

Generate optimized AGENTS.md files for AI coding agents.

## Workflow Checklist

Copy and check off as you progress:

- [ ] **1. Gather Context:** Run `scripts/gather-context.sh [project-path]`
- [ ] **2. Choose Template:** Select from `templates/` based on project type
- [ ] **3. Generate:** Copy template to project root as `AGENTS.md`
- [ ] **4. Customize:** Add project-specific details
- [ ] **5. Validate:** Run `scripts/validate-agents-md.sh AGENTS.md`
- [ ] **6. Fix Issues:** Address errors from validation
- [ ] **7. Re-validate:** Repeat until validation passes ⚠️
- [ ] **8. Commit:** Add AGENTS.md to version control

**⚠️ Only proceed to next step when validation passes.**

## Quick Start

```bash
# 1. Gather project context
./scripts/gather-context.sh /path/to/project

# 2. Use standard template
cp templates/standard.md /path/to/project/AGENTS.md

# 3. Edit with project details
vim /path/to/project/AGENTS.md

# 4. Validate
./scripts/validate-agents-md.sh /path/to/project/AGENTS.md

# 5. Fix errors and re-validate until it passes
```

## Validation Loop Pattern

This skill enforces a strict validation loop:

```
Generate → Validate → Fix → Validate → Repeat until ✅
```

**Never skip validation.** AI agents perform better with validated AGENTS.md files.

## Templates

| Template | Words | Lines | Best For |
|----------|-------|-------|----------|
| `minimal.md` | 50-80 | <60 | Small projects, quick setup |
| `standard.md` | 100-150 | <150 | Most projects |
| `monorepo-root.md` | 80-120 | <100 | Multi-package root |
| `monorepo-package.md` | 60-100 | <80 | Package-specific |

**Selection guide:**
- Single project with <10 files → `minimal.md`
- Single project with >10 files → `standard.md`
- Monorepo root → `monorepo-root.md`
- Monorepo package → `monorepo-package.md`

## Scripts

### gather-context.sh

Detects project metadata:
- Structure (monorepo vs single)
- Package manager (npm/yarn/pnpm/bun/pip/cargo/go)
- Tech stack (Node.js/Python/Go/Rust)
- CI/CD (GitHub Actions/GitLab CI/Jenkins)
- Existing AI configs (.cursorrules/CLAUDE.md)

```bash
./scripts/gather-context.sh [project-path]
```

**Output:** Prints context to terminal (optionally save to `.agents-md-context.json`)

### interview.sh

Interactive prompts for customization:
- Primary tasks (feature dev, bug fixes, refactoring)
- Code style preferences
- Testing requirements
- Safety boundaries
- Dos and Don'ts

```bash
./scripts/interview.sh              # Interactive
./scripts/interview.sh --non-interactive  # Use defaults
```

### validate-agents-md.sh

Validates against best practices:
- ✅ Length (335-535 words optimal)
- ✅ Required sections (Dev Setup, Build, Test, Code Style)
- ✅ No AI vocabulary triggers
- ✅ Standard Markdown only
- ✅ Actionable error messages

```bash
./scripts/validate-agents-md.sh AGENTS.md
```

**Exit codes:**
- `0` = Pass (or pass with warnings)
- `1` = Fail (errors must be fixed)

## Best Practices

### Length Guidelines

Research shows AI agents follow ~150-200 instructions effectively.

- **Optimal:** 335-535 words
- **Maximum:** 800 words (hard limit)
- **Reason:** Longer files get ignored

**If too long:** Move detailed docs to `references/` folder.

### Progressive Disclosure

```
your-project/
├── AGENTS.md          # Universal rules (<150 lines)
└── references/
    ├── api-guide.md   # API-specific details
    ├── testing.md     # Testing patterns
    └── style.md       # Detailed style guide
```

**Rule:** Keep references one level deep only. Agents struggle with multi-hop navigation.

### Required Sections

Every AGENTS.md should have:

1. **Project overview** - One sentence + tech stack
2. **Dev Setup** - Install, run, build commands
3. **Build & Test** - How to verify changes
4. **Code Style** - Essential conventions only
5. **Testing** - Run tests, coverage requirements

### What to Include

✅ **Do include:**
- Specific commands (`pnpm install`, `npm test`)
- Project-specific conventions
- Non-obvious gotchas
- Architecture decisions
- Tool preferences

❌ **Don't include:**
- Basic concepts (what is TypeScript)
- Generic advice (write clean code)
- Auto-generated content
- Tool-specific syntax (`@path`, `{{template}}`)

## Examples

Real AGENTS.md files for reference:

- `references/examples/node-api.md` - Node.js REST API (92 words)
- `references/examples/react-frontend.md` - React dashboard (78 words)
- `references/examples/go-cli.md` - Go CLI tool (87 words)

## Validation Patterns

### Common Errors & Solutions

| Error | Solution |
|-------|----------|
| Too long (>800 words) | Move API docs to `references/api-guide.md` |
| Missing: Dev Setup | Add `## Dev Setup` with install command |
| Missing: Testing | Add `## Testing` with test command |
| AI vocabulary detected | Replace "delve" → "explore", "crucial" → "important" |
| Tool-specific syntax | Remove `@path`, use standard Markdown links |

### Error Message Format

Validation errors are actionable:

```
❌ Missing: Testing
   Add: ## Testing
        - Run: npm test
        - Coverage: npm run test:coverage
```

**Not:** "Validation failed" (vague, unhelpful)

## Compatibility

Works with all AI coding agents:
- Claude Code
- Codex
- Cursor
- Aider
- OpenCode
- Windsurf
- Any agent that reads project context

## Research Base

Built on research from:
- [agents.md](https://agents.md/) - 60k+ projects analyzed
- [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md) - AI instruction limits
- [Builder.io](https://www.builder.io/blog/agents-md) - Best practices
- Skill creation patterns - Progressive disclosure, validation loops

See `references/best-practices.md` for full synthesis.

## References

- `best-practices.md` - Complete research synthesis
- `section-guide.md` - When to use each section
- `template-guide.md` - Customizing templates
- `migration-guide.md` - Migrate from .cursorrules, etc.
- `patterns.md` - Common AGENTS.md patterns
- `agent-quirks.md` - Tool-specific behaviors
- `workflow-patterns.md` - Validation loops, checklists
- `examples/` - Real AGENTS.md files

---
> Source: [goncalovelosa/agents-md-creator](https://github.com/goncalovelosa/agents-md-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
