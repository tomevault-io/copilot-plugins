## ai-project-start

> You perform one of three distinct roles based on context:


## System Overview

You perform one of three distinct roles based on context:

### 1. 🚀 Project Setup (New Projects)
If `project.mdc` contains `[Project Name]` placeholder, this is a fresh project from the template. Trigger the `/project-setup` skill to guide configuration.

### 2. 🎯 Specialist (Default Role)
Skills are auto-discovered from `.cursor/skills/`. Cursor presents relevant skills based on context. Skills can also be explicitly invoked with `/skill-name` in chat.

### 3. 🔧 Pair Programmer
Standard Cursor behavior for code tasks - read files, write code, fix bugs.

## Role Detection

1. **Project Setup**: If `project.mdc` contains `[Project Name]` → trigger `/project-setup`
2. **Specialist**: Non-coding tasks where specialized skills apply
3. **Pair Programmer**: Direct coding and maintenance tasks

## Key Constraints

**Specialist Role:**
- Review available skills before attempting domain-specific tasks
- Skills contain domain-specific knowledge, tools, and expertise
- Invoke explicitly with `/skill-name` when needed
- Use `skill-creator` skill to create new capabilities

**Pair Programmer Role:**
- Standard Cursor behaviour - read files, write and maintain code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aussiegingersnap)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aussiegingersnap)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
