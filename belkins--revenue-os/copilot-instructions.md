## revenue-os

> Revenue OS is a Claude Code plugin that helps developers monetize their products. It provides a systematic approach to go from code to customers to cash.

# Revenue OS - Development Context

## Project Overview

Revenue OS is a Claude Code plugin that helps developers monetize their products. It provides a systematic approach to go from code to customers to cash.

**Tagline**: "From code to customers to cash"

## Architecture

```
revenue-os/
├── .claude-plugin/
│   └── plugin.json         # Plugin manifest with command definitions
├── commands/               # Slash commands (/ros, /ros icp, etc.)
│   ├── ros.md              # Main dashboard
│   ├── ros-icp.md          # ICP discovery
│   ├── ros-value-prop.md   # Value proposition
│   ├── ros-pricing.md      # Pricing architecture
│   ├── ros-competitors.md  # Competitive intelligence
│   ├── ros-outreach.md     # Outreach sequences
│   ├── ros-landing-page.md # Landing page generator
│   ├── ros-objections.md   # Objection handling
│   ├── ros-first-dollar.md # Zero to revenue playbook
│   └── ros-audit.md        # Revenue readiness audit
├── data/                   # JSON data files (templates, frameworks)
│   ├── icp-templates.json
│   ├── value-prop-frameworks.json
│   ├── pricing-psychology.json
│   ├── competitor-analysis.json
│   ├── outreach-templates.json
│   ├── objection-library.json
│   └── industry-benchmarks.json
├── hooks/
│   └── hooks.json          # Session lifecycle hooks
├── scripts/
│   ├── core/
│   │   └── init-session.sh # Initialize data directories
│   └── utils/
│       └── storage.sh      # Data persistence utilities
├── CLAUDE.md               # This file
└── README.md               # User documentation
```

## Data Flow

Commands build on each other in this recommended order:

1. `/ros icp` analyzes codebase → saves to `~/.claude/revenue-os/icp.json`
2. `/ros value-prop` uses ICP data → saves to `~/.claude/revenue-os/value-prop.json`
3. `/ros pricing` uses ICP + competitor research → saves to `~/.claude/revenue-os/pricing.json`
4. `/ros outreach` uses ICP + value prop → generates sequences in `~/.claude/revenue-os/outreach/`
5. `/ros audit` reads all data → provides comprehensive score

## Data Storage

All persistent data stored in: `~/.claude/revenue-os/`

| File | Purpose |
|------|---------|
| `product.json` | Product profile from codebase analysis |
| `icp.json` | Ideal customer profiles (primary, secondary) |
| `value-prop.json` | Value propositions and messaging |
| `pricing.json` | Pricing tiers and strategy |
| `outreach/` | Generated outreach sequences |
| `exports/` | Exported data backups |

## Command Development

### Creating a New Command

1. Create `commands/ros-[name].md`
2. Add YAML frontmatter:
   ```yaml
   ---
   description: Brief description of command
   argument-hint: [arg1|arg2|arg3]
   allowed-tools: Read, Glob, Grep, WebSearch, Bash(cat:*), Write
   ---
   ```
3. Add data loading section using `!` bash commands
4. Define instructions for each argument case
5. Specify output format
6. Add save instructions if data should persist

### Command Template

```markdown
---
description: Command description
argument-hint: [option1|option2]
allowed-tools: Read, Bash(cat:*), Write
---

# Command Name

## Arguments: $ARGUMENTS

## Load Data

!`cat ~/.claude/revenue-os/product.json 2>/dev/null || echo '{"name": null}'`
!`cat "${CLAUDE_PLUGIN_ROOT}/data/relevant-data.json" 2>/dev/null || echo '{}'`

## Instructions

### If $ARGUMENTS is empty or "default":
[Default behavior]

### If $ARGUMENTS is "option1":
[Option 1 behavior]

## Output Format

[Specify expected output structure]

## Save Results

[Instructions for persisting data]
```

## Testing

### Manual Testing Checklist

1. **Dashboard**: `/ros` should show score and checklist
2. **ICP Discovery**: `/ros icp` should analyze current codebase
3. **ICP Validation**: `/ros icp validate` should use web search
4. **Value Prop**: `/ros value-prop` should use ICP data
5. **Pricing**: `/ros pricing` should research competitors
6. **Outreach**: `/ros outreach` should generate personalized sequences
7. **Audit**: `/ros audit` should provide accurate scoring

### Testing Data Persistence

```bash
# Check if data directory exists
ls -la ~/.claude/revenue-os/

# Verify JSON files are valid
cat ~/.claude/revenue-os/product.json | jq '.'
cat ~/.claude/revenue-os/icp.json | jq '.'

# Check outreach sequences
ls ~/.claude/revenue-os/outreach/
```

## Key Design Principles

1. **Analyze YOUR code**: Don't give generic advice - understand the specific product
2. **Use web research**: Real competitor data, real community discovery
3. **Actionable outputs**: Every command produces something usable TODAY
4. **Connected system**: ICP informs value prop, which informs outreach
5. **Engineering mindset**: Revenue as a system to be designed and debugged
6. **Persistent state**: Remember product, ICP, value prop across sessions

## Common Issues

### Data Not Persisting
- Check if `~/.claude/revenue-os/` directory exists
- Verify init-session.sh has execute permissions
- Check for JSON syntax errors in saved files

### Web Search Not Working
- Ensure `WebSearch` is in allowed-tools
- Check if command is using WebSearch correctly

### Command Not Found
- Verify command file exists in `commands/` directory
- Check plugin.json includes the command
- Ensure YAML frontmatter is valid

## Contributing

When adding new features:
1. Follow existing command patterns
2. Add data files to `data/` directory
3. Update this CLAUDE.md with architecture changes
4. Update README.md with user-facing changes
5. Test all commands after changes

---
> Source: [Belkins/revenue-os](https://github.com/Belkins/revenue-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
