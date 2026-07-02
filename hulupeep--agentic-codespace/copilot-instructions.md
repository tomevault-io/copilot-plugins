## agentic-codespace

> This is an Agentic Codespace with tide data for building marine apps. Claude Code and Claude Flow are pre-configured.

# Claude Code Configuration

## Project Overview
This is an Agentic Codespace with tide data for building marine apps. Claude Code and Claude Flow are pre-configured.

## Quick Commands

### Claude Code (Your AI Assistant)
```bash
# Basic usage
claude "what files are in this project?"
claude "show me the tide_times database"
claude "build a swimming safety dashboard"
```

### Claude Flow (Multi-Agent Orchestration)
```bash
# SPARC methodology
npx claude-flow sparc modes                    # List all modes
npx claude-flow sparc run architect "design API"  # Run specific mode
npx claude-flow sparc tdd "user auth"         # Test-driven development
```

## Available Data
- **Database**: 366 days of Galway tide times
- **APIs**: ERDDAP marine data (waves, temperature, weather)
- **Examples**: Ready-to-run code in `/examples`

## Project Structure
```
/examples     - Working examples
/tidedata     - Marine data project files  
/supabase     - Database configuration
/.devcontainer - Codespace setup
```

## Development Workflow

1. **Explore**: `claude "analyze this codebase"`
2. **Plan**: `npx claude-flow sparc run architect "your feature"`
3. **Build**: `npx claude-flow sparc tdd "implement feature"`
4. **Test**: `npm test`

## Important Notes
- Claude Code works with your subscription OR API key
- Supabase Studio: http://localhost:54323
- Check setup: `./check-setup.sh`
- Update tools: `./update-tools.sh`

---
> Source: [Hulupeep/Agentic_Codespace](https://github.com/Hulupeep/Agentic_Codespace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
