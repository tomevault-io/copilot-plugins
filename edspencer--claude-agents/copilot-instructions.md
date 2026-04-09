## claude-agents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin marketplace** containing specialized agent teams for software development workflows. Each plugin in the `plugins/` directory represents a complete, self-contained agent team with its own agents, commands, skills, and documentation.

The repository follows the **Anthropic claude-code plugin marketplace pattern**, allowing users to install individual plugins from a single marketplace.

## Repository Structure

```
claude-agents/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (defines available plugins)
├── plugins/
│   └── product-team/             # Product development workflow plugin
│       ├── .claude-plugin/
│       │   ├── plugin.json       # Plugin metadata (simple schema)
│       │   └── marketplace.json  # Standalone plugin marketplace manifest
│       ├── agents/               # 11 specialized agents
│       ├── commands/             # 14 slash commands
│       ├── skills/
│       │   └── github-task-sync/ # GitHub issue synchronization
│       ├── docs/
│       │   ├── standing-orders.md
│       │   ├── team.md
│       │   └── processes/        # Process documentation (6 files)
│       └── README.md
├── README.md                     # Marketplace documentation
├── LICENSE                       # MIT License
└── .gitignore
```

## Key Concepts

### Multi-Plugin Marketplace

This repository is a **marketplace** containing multiple plugins. Users add the marketplace once and install individual plugins:

```bash
/plugin marketplace add edspencer/claude-agents
/plugin install product-team@edspencer-agents
```

The marketplace name is `edspencer-agents` (from `.claude-plugin/marketplace.json`), which is unique and avoids naming collisions.

### Plugin Schema Requirements

**Important:** Plugin manifests have strict schemas. If you modify them, ensure:

#### Root Marketplace Manifest (`.claude-plugin/marketplace.json`)
```json
{
  "name": "edspencer-agents",           // Marketplace identifier (kebab-case)
  "version": "1.0.0",
  "owner": {
    "name": "Ed Spencer",
    "email": "ed@edspencer.net"
  },
  "plugins": [
    {
      "name": "product-team",           // Plugin identifier
      "description": "...",
      "version": "0.1.0",
      "author": {                       // MUST be object with name/email
        "name": "Ed Spencer",
        "email": "ed@edspencer.net"
      },
      "source": "./plugins/product-team",
      "category": "Development"
    }
  ]
}
```

**Key Rules:**
- `author` MUST be an object with `name` and `email`
- Do NOT include `displayName` field (causes validation error)
- `source` is relative path to plugin directory

#### Individual Plugin Manifest (`plugins/*/. claude-plugin/plugin.json`)
```json
{
  "name": "product-team",
  "version": "0.1.0",
  "description": "...",
  "author": {                           // MUST be object
    "name": "Ed Spencer",
    "email": "ed@edspencer.net"
  }
}
```

**Key Rules:**
- ONLY include: `name`, `version`, `description`, `author`
- Do NOT include: `displayName`, `tags`, `agents`, `commands`, `skills`, `hooks`, `mcpServers` (these cause validation errors)
- Keep it minimal - Claude Code infers capabilities from directory structure

### Layered Documentation Pattern

Plugins use a **layered configuration system**:

**Layer 1: Plugin Defaults (Global)**
- Location: `~/.claude/plugins/marketplaces/edspencer-agents/plugins/product-team/docs/`
- Provides base process rules that work across projects

**Layer 2: Project Overrides (Local)**
- Location: `.claude/docs/` (in user's project)
- Project-specific extensions or overrides

Agents check both locations:
1. Check `.claude/docs/[file].md` in project (if exists)
2. Fall back to plugin path

**Known Issue:** Due to [Claude Code issue #9354](https://github.com/anthropics/claude-code/issues/9354), `${CLAUDE_PLUGIN_ROOT}` doesn't work in agent/command markdown files. We use hardcoded paths: `~/.claude/plugins/marketplaces/edspencer-agents/plugins/product-team/` as a workaround.

## Product Team Plugin

The first and currently only plugin in this marketplace.

### Components

**11 Agents:**
- Specification: spec-writer, spec-checker
- Planning: plan-writer, plan-checker
- Implementation: code-writer, code-checker
- Documentation: documentation-manager
- Testing: browser-tester
- Management: engineering-manager, process-manager, agent-maker

**14 Slash Commands:**
- `/write-spec`, `/check-spec`
- `/write-plan`, `/check-plan`
- `/write-code`, `/check-code`
- `/agentic-spec-and-plan`, `/agentic-create-spec`, `/agentic-create-plan`, `/agentic-implement-plan`
- `/run-integration-tests`, `/add-to-test-plan`
- `/finish`, `/create-issue`

**1 Skill:**
- github-task-sync: Bidirectional sync between local task files and GitHub issues

**6 Process Documents:**
- spec-rules.md, plan-rules.md, code-rules.md
- agent-rules.md, process-manager-rules.md
- Standing orders and team documentation

## Development Workflow

### Adding New Plugins

When adding a new plugin (e.g., `data-team`, `design-team`):

1. **Create plugin directory:**
   ```bash
   mkdir -p plugins/new-team/.claude-plugin
   mkdir -p plugins/new-team/{agents,commands,skills,docs}
   ```

2. **Create plugin.json:**
   ```json
   {
     "name": "new-team",
     "version": "0.1.0",
     "description": "...",
     "author": {
       "name": "Ed Spencer",
       "email": "ed@edspencer.net"
     }
   }
   ```

3. **Add to marketplace.json:**
   Add entry to `.claude-plugin/marketplace.json` plugins array

4. **Create README.md:**
   Document the plugin's agents, commands, and usage

5. **Test locally:**
   ```bash
   cp -r . ~/.claude/plugins/repos/claude-agents
   # Restart Claude Code
   /plugin install new-team@edspencer-agents
   ```

### Modifying Existing Plugins

When updating the product-team plugin:

1. **Update version:** Increment version in both:
   - `.claude-plugin/marketplace.json` (plugin entry)
   - `plugins/product-team/.claude-plugin/plugin.json`

2. **Update documentation:**
   - Plugin README with new features
   - Root README if adding significant capabilities

3. **Update agent counts:**
   If adding/removing agents or commands, update counts in both READMEs

### Testing Changes

```bash
# Test locally before pushing
cd ~/Code/claude-agents
cp -r . ~/.claude/plugins/repos/claude-agents

# Restart Claude Code
# Then verify plugin loads without errors
/plugin
```

Look for "Plugin Loading Errors" section. If present, check:
- Marketplace.json schema (author as object, no displayName)
- Plugin.json schema (minimal fields only)

## Installation Instructions

For users of this marketplace:

```bash
# Add the marketplace
/plugin marketplace add edspencer/claude-agents

# Install Product Team plugin
/plugin install product-team@edspencer-agents

# Verify installation
/plugin
```

The plugin will be installed to:
```
~/.claude/plugins/marketplaces/edspencer-agents/plugins/product-team/
```

## Future Plugins

Planned plugins to add to this marketplace:

- **data-team**: Data engineering, analytics, ETL workflows
- **design-team**: Design system, UI/UX, component library workflows
- **infrastructure-team**: DevOps, deployment, monitoring workflows
- **qa-team**: Quality assurance, test automation, bug tracking workflows

Each plugin will follow the same structure and patterns as product-team.

## Key Files to Modify

When working with this repository, you'll typically modify:

### Adding Plugin to Marketplace
- `.claude-plugin/marketplace.json` - Add plugin entry
- `README.md` - Document new plugin
- `plugins/new-plugin/` - Create plugin directory

### Updating Plugin Metadata
- `plugins/*/. claude-plugin/plugin.json` - Plugin version, description
- `plugins/*/README.md` - Plugin documentation

### Adding Agents/Commands
- `plugins/*/agents/*.md` - New agent definitions
- `plugins/*/commands/*.md` - New slash commands
- `plugins/*/docs/team.md` - Update agent roster
- `plugins/*/README.md` - Update counts and descriptions

### Adding Process Documentation
- `plugins/*/docs/processes/*.md` - New process rules
- Update agents that reference new processes

## Common Patterns

### Agent Development Pattern

1. **Create agent file** in `plugins/product-team/agents/[name].md`
2. **Include frontmatter** with name, description, model, color
3. **Add documentation lookup section** referencing layered docs
4. **Define responsibilities and workflows**
5. **Reference process docs** from `docs/processes/`
6. **Update team.md** with agent description

### Command Development Pattern

1. **Create command file** in `plugins/product-team/commands/[name].md`
2. **Include frontmatter** with allowed-tools, argument-hint, description
3. **Define task and requirements**
4. **Reference relevant process docs**
5. **Update README** with command in appropriate section

### Process Documentation Pattern

1. **Create process doc** in `plugins/product-team/docs/processes/[name].md`
2. **Define clear rules and guidelines**
3. **Include examples and patterns**
4. **Update agents** that should follow this process
5. **Note in README** if major process added

## Contributing

When contributing to this repository:

1. **Follow existing patterns** for consistency
2. **Test schema validation** before committing manifest changes
3. **Update documentation** when adding features
4. **Maintain version numbers** appropriately
5. **Keep plugins self-contained** - no cross-plugin dependencies

## License

MIT License - see LICENSE file

## Related Resources

- [Claude Code Documentation](https://code.claude.com/docs)
- [Plugin Development Guide](https://code.claude.com/docs/en/plugins)
- [Plugin Marketplaces Guide](https://code.claude.com/docs/en/plugin-marketplaces)
- [Agent Skills Documentation](https://code.claude.com/docs/en/skills)
- [Claude Code GitHub Issue #9354](https://github.com/anthropics/claude-code/issues/9354) - CLAUDE_PLUGIN_ROOT bug

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edspencer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/edspencer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
