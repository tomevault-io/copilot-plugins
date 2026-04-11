## homebrew-mcpops

> This is a Homebrew formula repository for mcpops - a cross-platform MCP configuration management tool.

# AGENTS.md - Development Guidelines

## Repository Overview
This is a Homebrew formula repository for mcpops - a cross-platform MCP configuration management tool.

## Build/Test Commands
- **Test formula**: `brew install --build-from-source ./mcpops.rb`
- **Validate formula**: `brew audit --strict mcpops.rb`
- **Test installation**: `brew test mcpops` (if test block exists)
- **Lint formula**: `brew style mcpops.rb`

## Code Style Guidelines

### Ruby/Homebrew Formula Style
- Use 2-space indentation (no tabs)
- Follow Homebrew formula conventions and naming
- Use double quotes for strings containing interpolation, single quotes otherwise
- Class names should match formula name in PascalCase
- Use `desc` for short description, `homepage` for project URL
- Always include `url`, `sha256`, and `version`
- Use `prefix.install` for app bundles, `bin.install` for executables
- Keep formula concise and focused on installation logic

### Error Handling
- Let Homebrew handle standard installation errors
- Add custom error messages only for formula-specific issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mcp-ops)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mcp-ops)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
