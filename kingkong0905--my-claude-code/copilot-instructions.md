## my-claude-code

> This is the `kkclaude` Claude Code plugin repository. Follow these instructions when developing or modifying the plugin.

# Plugin Development Instructions

This is the `kkclaude` Claude Code plugin repository. Follow these instructions when developing or modifying the plugin.

## Project Structure

- **`.claude-plugin/`** — Plugin manifest (`plugin.json`) and marketplace metadata
- **`agents/`** — Agent definitions in markdown with YAML frontmatter
- **`skills/`** — Skill definitions (slash commands + domain knowledge)
- **`hooks/`** — Lifecycle hook scripts (`hooks.json` + `scripts/`)

## Development Conventions

### Adding a New Skill

1. Create `skills/<name>/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   description: [What this skill does]
   argument-hint: "[argument description]"
   model: haiku | sonnet | opus
   allowed-tools: [comma-separated list]
   ---
   ```
2. Write clear, step-by-step instructions in the body.
3. Keep skills focused — one skill, one job.

### Adding a New Agent

1. Create `agents/<name>.md` with YAML frontmatter:
   ```yaml
   ---
   name: kkclaude:<name>
   description: [What this agent does]
   model: haiku | sonnet | opus
   allowed-tools: [comma-separated list]
   ---
   ```
2. Use `haiku` for cheap, fast tasks; `sonnet` for complex tasks.

### Adding a Hook Script

1. Add the script to `hooks/scripts/<name>.js`
2. Register it in `hooks/hooks.json`
3. Hook scripts must:
   - Use ESM (`import`/`export`)
   - Exit with code `0` to allow, `1` to block (PreToolUse only)
   - Be fast (< 1 second) — they run on every tool call

### Code Style

- Node.js ESM (`"type": "module"` in package.json)
- ES2024 features allowed
- Run `npm run lint` before committing

## Testing Changes

```bash
# Lint hook scripts
npm run lint

# Test a specific hook manually
echo '{"tool_input":{"command":"echo hello"}}' | node hooks/scripts/bash-safety-check.js
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kingkong0905) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
