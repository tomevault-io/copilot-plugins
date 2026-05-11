## ace-claude-code

> **Ace** is a unified multi-agent orchestration system for Claude Code.

# Ace

**Ace** is a unified multi-agent orchestration system for Claude Code.

## Activation

To activate Ace, use one of these methods:

1. **Say "Ace"** - Auto-detect project type and start
2. **Use slash commands** - `/ace:help` for full list

## Quick Start

### New Project (Greenfield)
```
/ace:new-project "My App"
/ace:create-roadmap
/ace:plan-phase 1
/ace:execute-plan
```

### Existing Project (Brownfield)
```
/ace:map-codebase
/ace:new-project "My App"
# Continue as above...
```

### Resume Session
```
/ace:progress      # See where you are
/ace:resume-work   # Restore context
```

## Key Commands

| Command | Description |
|---------|-------------|
| `/ace:new-project` | Initialize project |
| `/ace:create-roadmap` | Plan phases |
| `/ace:plan-phase [N]` | Create tasks |
| `/ace:execute-plan` | Run with parallel agents |
| `/ace:progress` | Check status |
| `/ace:help` | All commands |

## Architecture

```
                   ORCHESTRATOR
                        |
        +---------------+---------------+
        |               |               |
     ALPHA           BETA            GAMMA
     Lead            Lead            Lead
        |               |               |
     mini            mini            mini
    agents          agents          agents
```

## Documentation

- `.ace/SKILL.md` - Full orchestration system
- `.ace/config.json` - Configuration
- `.ace/commands/*.md` - Command definitions
- `.ace/templates/` - Document templates

---

Load the skill: Read `.ace/SKILL.md` to activate full Ace capabilities.

---
> Source: [AgriciDaniel/ace-claude-code](https://github.com/AgriciDaniel/ace-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
