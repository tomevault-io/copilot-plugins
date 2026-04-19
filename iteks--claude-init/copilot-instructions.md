## claude-init

> A Claude Code skill that analyzes any project and generates optimal Claude Code integration — hooks, rules, agents, skills, commands, settings, and CLAUDE.md.

# claude-init

A Claude Code skill that analyzes any project and generates optimal Claude Code integration — hooks, rules, agents, skills, commands, settings, and CLAUDE.md.

## Stack

- Shell scripts (bash) + JSON + Markdown
- No build system, no runtime dependencies beyond bash + jq
- Distribution: Git clone + symlink to `~/.claude/skills/`

## Architecture

### 5-Phase Pipeline

claude-init operates as a single SKILL.md file (`skills/claude-init/SKILL.md`) with a template library. The pipeline:

1. **Phase 1 — Detect State**: Determine mode (New / Existing / Audit) based on `.claude/` and source code presence. The `global` subcommand has its own pipeline (Phases G1–G5) for configuring `~/.claude/`.
2. **Phase 2 — Analyze**: Auto-detect stack (language, framework, test runner, formatter, package manager, dev commands, infrastructure) or prompt the user for new projects. Suggest agents, skills, plugins, and commands based on project signals.
3. **Phase 3 — Generate**: Read templates, replace placeholders, write config files. Generate permissions from detected tools.
4. **Phase 4 — Validate**: Verify hook executability, JSON validity, rule path matches, CLAUDE.md line count
5. **Phase 5 — Report**: Summarize everything created, provide next steps including plugin install commands

### Subagent Delegation

Heavy phases are delegated to subagents to preserve the main session's context:
- **Detection subagent** (haiku): Phases 2B + 2C-2E — file scanning, convention extraction, signal detection
- **Generation subagent** (sonnet): Phases 3 + 4 — template reading, placeholder replacement, file writing, validation
- **Main session**: Phase 1 (lightweight checks), user interaction (prompts, confirmations), Phase 5 (summary display)

### Template Library

All templates live in `skills/claude-init/templates/`:

| Directory | Purpose | Count |
|---|---|---|
| `global/` | Global `~/.claude/` templates (CLAUDE.md, agents, commands, rules, memory) | 10 |
| `settings/` | `.claude/settings.json` templates per framework | 9 |
| `hooks/` | Hook scripts organized by language (`universal/`, `php/`, `javascript/`, `python/`, `go/`, `rust/`, `shell/`) | 12 |
| `rules/` | Path-scoped convention rules (`php/`, `javascript/`, `python/`) | 9 |
| `agents/` | Agent definitions (core + security + test + suggested) | 15 |
| `claude-md/` | CLAUDE.md templates per framework | 6 |
| `skills/` | Skill SKILL.md templates per framework | 5 |
| `commands/` | Slash command templates | 3 |

### Placeholder System

Templates use `{{PLACEHOLDER}}` syntax for values replaced during generation:
- `{{PROJECT_NAME}}`, `{{PROJECT_DESCRIPTION}}`, `{{PROJECT_SLUG}}` — from directory or user input
- `{{PHP_VERSION}}`, `{{LARAVEL_VERSION}}`, `{{PYTHON_VERSION}}`, `{{DJANGO_VERSION}}`, `{{FASTAPI_VERSION}}`, `{{NEXTJS_VERSION}}`, `{{REACT_VERSION}}`, `{{EXPO_VERSION}}` — from detected versions
- `{{DEV_COMMAND}}`, `{{TEST_COMMAND}}`, `{{FORMAT_COMMAND}}`, `{{BUILD_COMMAND}}`, `{{LINT_COMMAND}}`, `{{TYPECHECK_COMMAND}}` — from dev command detection
- `{{FORMATTER}}`, `{{FORMATTER_COMMAND}}` — from detected formatter
- `{{DATABASE}}`, `{{PACKAGE_MANAGER}}`, `{{INDENT_STYLE}}` — from project analysis
- `{{STYLING}}`, `{{STYLING_CONVENTIONS}}` — from detected CSS/styling framework
- `{{ARCHITECTURE_NOTES}}`, `{{CONVENTIONS}}`, `{{WATCH_ITEMS}}` — from source code analysis
- `{{STACK_DESCRIPTION}}`, `{{DIRECTORY_TABLE}}` — combined project summaries
- `{{DEV_INSTRUCTIONS}}` — generated from detected dev commands
- `{{DEV_URL}}` — from local dev environment detection
- `{{TEST_FRAMEWORK}}` — from test framework detection
- `{{RESOURCE_NAME}}`, `{{RESOURCE_PLURAL}}` — runtime, resolved at skill invocation from `$ARGUMENTS`
- `{{COMPONENT_DIR}}` — runtime, resolved at skill invocation time
- `{{APP_CONFIG_NAME}}` — runtime, derived from skill `$ARGUMENTS`

Global pipeline placeholders (from G2 user preference prompts):
- `{{CODING_STYLE}}`, `{{COMMUNICATION_TONE}}`, `{{INDENT_PREFERENCE}}`, `{{GIT_CONVENTIONS}}`, `{{TOOL_PREFERENCES}}`

## Key Files

| File | Purpose |
|---|---|
| `skills/claude-init/SKILL.md` | Core skill definition (~1350 lines) — project pipeline + global pipeline |
| `install.sh` | Symlinks skill globally + merges permission optimizations |
| `uninstall.sh` | Removes symlink + cleans up settings |
| `global/settings-patch.json` | Global permission pre-approvals and denials |
| `global/check-update.sh` | SessionStart hook — two-tier update detection (local + throttled remote) |
| `global/self-update.sh` | Self-update script — fetches latest tag and checks out |

## Conventions

- **Indentation**: 2 spaces for JSON/Markdown/Shell scripts
- **Hook scripts**: Always check `jq` availability first (`if ! command -v jq &>/dev/null; then exit 0; fi`), read stdin with `INPUT=$(cat)`, structured output with `jq -n`
- **Agent frontmatter fields**: `name`, `description`, `model`, `color`, `tools`, `permissionMode`, `maxTurns`, `memory`
- **Rule frontmatter**: `paths:` array with glob patterns — always validate globs match actual files
- **Skill frontmatter**: `description` (required), optionally `disable-model-invocation: true`
- **Command frontmatter**: `description` (required), optionally `disable-model-invocation: true`
- **Template naming**: `{type}-{framework}.{ext}` (e.g., `security-reviewer-php.md`, `php-laravel.json`)
- **CLAUDE.md limit**: All generated CLAUDE.md files must stay under 300 lines

## Workflow Automation

### Task Assessment

Use `EnterPlanMode` for any task that:
- Modifies SKILL.md (the core ~1350-line pipeline)
- Adds a new stack (requires templates + detection logic + template selection table)
- Changes template conventions that affect multiple files

### Post-Implementation

After modifying 2+ files:
- Verify SKILL.md phase numbering is consistent
- Verify template selection table covers new outputs
- Check that new templates follow existing patterns (hook structure, agent frontmatter, rule format)
- Run `wc -l skills/claude-init/SKILL.md` — should be ~1350 lines

### Context Management

- After completing a unit of work, suggest `/compact`
- After 5+ exchanges on different topics, suggest `/clear`

## Things to Watch For

- **SKILL.md is the single source of truth** — all pipeline logic lives there, not in external scripts
- **Template patterns must be consistent** — hooks use jq stdin pattern, agents use correct frontmatter fields, rules validate their path globs
- **Placeholder replacement** — every `{{PLACEHOLDER}}` in a template must have a corresponding replacement in the Phase 3 generation instructions
- **Phase numbering** — after 2B (detection), phases are 2C (agents), 2D (skills), 2E (plugins/commands), 2F (audit)
- **Line count discipline** — generated CLAUDE.md files must stay under 300 lines; excess content goes to path-scoped rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iteks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
