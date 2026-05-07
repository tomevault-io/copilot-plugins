## audio-plugin-dev-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code marketplace plugin providing AI-assisted skills for validating audio plugins across formats (VST3, AudioUnit, CLAP). This is a documentation-driven project — skills are Markdown files that give Claude context for running external validator CLI tools.

## Repository Structure

```
.claude-plugin/marketplace.json         # Marketplace registration
plugins/audio-plugin-validators/
  .claude-plugin/plugin.json            # Plugin metadata (name, version, keywords)
  skills/
    validate-audiounit/SKILL.md         # auval (macOS only)
    validate-clap/SKILL.md              # clap-validator
    validate-pluginval/SKILL.md         # pluginval (VST3/AU)
    validate-vst3/SKILL.md              # Steinberg validator
    validate-vst3-editorhost/SKILL.md   # Steinberg editorhost
    check-codesign/SKILL.md             # macOS code signature check
```

## Adding a New Skill

1. Create `plugins/audio-plugin-validators/skills/<skill-name>/SKILL.md`
2. Include YAML frontmatter with: `name`, `description` (with trigger phrases), `compatibility`, and `metadata` (author, version)
3. Structure the body with: Instructions, Basic Usage, Options, Common Issues sections
4. Register the skill by adding it — auto-discovery finds `SKILL.md` files in skill directories

## SKILL.md Conventions

- The `description` field doubles as the trigger — include natural phrases users would say (e.g., "validate VST3", "test the .vst3")
- Provide platform-specific binary paths in tables
- Include installation/build instructions for the underlying CLI tool
- Document common failure modes and fixes
- Use code blocks with `bash` syntax highlighting

## Marketplace Metadata

- `marketplace.json` at `.claude-plugin/` lists all plugins with name, source path, description, and category
- Each plugin has its own `plugin.json` at `plugins/<name>/.claude-plugin/`
- `pluginRoot` in marketplace.json points to `./plugins`

## License

Copyright (c) 2026 Oli Larkin

---
> Source: [iPlug3/audio-plugin-dev-skills](https://github.com/iPlug3/audio-plugin-dev-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
