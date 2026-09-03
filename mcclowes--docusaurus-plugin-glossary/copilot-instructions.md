## docusaurus-plugin-glossary

> This document describes the specialized Claude Code agents and skills available for this project. These agents provide context-aware assistance for working with the docusaurus-plugin-glossary codebase.

# Claude Code Agents & Skills

This document describes the specialized Claude Code agents and skills available for this project. These agents provide context-aware assistance for working with the docusaurus-plugin-glossary codebase.

## What are Claude Skills?

Claude skills are specialized agents that provide domain-specific knowledge and assistance. When invoked, they load targeted context and instructions to help with specific tasks related to the project.

## Available Skills

### 1. `docusaurus-glossary`

**When to use:** Working with docusaurus-plugin-glossary configuration, managing glossary terms, troubleshooting issues, or explaining features.

**What it helps with:**

- Configuring the plugin in `docusaurus.config.js`
- Creating and managing glossary JSON files
- Troubleshooting auto-linking issues
- Understanding component usage (`<GlossaryTerm>`)
- Debugging term detection and tooltip behavior

**Example use cases:**

- "Why aren't my glossary terms auto-linking?"
- "How do I add a new glossary term?"
- "Configure the glossary plugin with custom route path"

### 2. `docusaurus-plugin-dev`

**When to use:** Developing and building Docusaurus plugins, especially understanding the plugin architecture used in this project.

**What it helps with:**

- Using `getClientModules()` for automatic DOM enhancement
- Implementing lifecycle hooks (`loadContent`, `contentLoaded`, etc.)
- SSR safety with `ExecutionEnvironment.canUseDOM`
- Integrating third-party libraries
- Understanding client vs server code separation

**Example use cases:**

- "How do I add a new client module?"
- "Implement a new lifecycle hook for the plugin"
- "Fix SSR errors in the plugin"

### 3. `docusaurus-plugins`

**When to use:** Creating or modifying Docusaurus plugins, particularly remark/rehype plugins for markdown/HTML transformation.

**What it helps with:**

- Creating remark plugins for markdown transformation
- Building rehype plugins for HTML processing
- Understanding AST traversal with `unist-util-visit`
- Implementing lifecycle plugins for routes/webpack
- Theme plugins and component swizzling

**Example use cases:**

- "Create a new remark plugin for custom syntax"
- "Modify the glossary-terms remark plugin"
- "Add a new theme component"

### 4. `docusaurus-config`

**When to use:** Validating or modifying `docusaurus.config.js` or `docusaurus.config.ts` files.

**What it helps with:**

- Plugin configuration
- Preset configuration
- Remark/rehype plugin setup
- Docusaurus configuration best practices

**Example use cases:**

- "Validate my docusaurus.config.js"
- "Add the glossary plugin to the config"
- "Configure remark plugins in the preset"

### 5. `docusaurus-themes`

**When to use:** Swizzling Docusaurus theme components or editing theme elements.

**What it helps with:**

- Component swizzling strategies
- Customizing theme components
- Understanding theme structure
- Overriding default styling

**Example use cases:**

- "Swizzle the GlossaryTerm component"
- "Customize the glossary page layout"
- "Override default theme styles"

## How to Use Skills

In Claude Code, you can invoke a skill by mentioning it in your conversation or by using the skill command:

```
Use the docusaurus-glossary skill to help me configure the plugin
```

or

```
/skill docusaurus-glossary
```

Skills automatically load relevant context and provide specialized assistance for the task at hand.

## Skill Architecture

This project uses the Claude Skills system which follows a progressive disclosure approach:

- **Level 1 (Discovery)**: Skill name and short description
- **Level 2 (Quick Reference)**: SKILL.md with core principles and quick start code
- **Level 3 (Deep Dive)**: Detailed reference documentation in `references/` directory

Each skill is located in `.claude/skills/<skill-name>/` and contains:

- `SKILL.md` - Quick reference and core principles
- `README.md` - Skill metadata and purpose
- `references/` - Detailed documentation files

## General Purpose Agents

In addition to specialized skills, Claude Code provides general-purpose agents:

### Explore Agent

**When to use:** Finding files, searching code, or understanding codebase structure.

**What it does:**

- Fast codebase exploration
- Pattern-based file finding
- Keyword code search
- Architectural understanding

**Usage:**

```
Explore the codebase to understand how remark plugins work
```

### Plan Agent

**When to use:** Planning complex multi-step implementations before coding.

**What it does:**

- Analyzes requirements
- Creates implementation plans
- Identifies dependencies
- Suggests architecture approaches

**Usage:**

```
Plan how to add support for glossary term aliases
```

## Best Practices

1. **Use the right skill for the task**: Each skill has specialized knowledge - use the most relevant one
2. **Combine skills when needed**: For complex tasks, you might need multiple skills
3. **Start with exploration**: Use the Explore agent to understand context before making changes
4. **Plan complex changes**: Use the Plan agent for multi-step implementations

## Contributing to Skills

Skills are stored in `.claude/skills/`. To add or modify a skill:

1. Create/edit the skill directory in `.claude/skills/<skill-name>/`
2. Update `SKILL.md` with core principles and quick reference
3. Keep `SKILL.md` concise (~50-150 lines)
4. Add detailed documentation to `references/` subdirectory
5. Ensure description in frontmatter is a single line

See individual skill files for examples and the progressive disclosure guidelines at the bottom of each `SKILL.md`.

## Related Documentation

- [README.md](README.md) - Plugin overview and usage
- [CONTRIBUTING.md](CONTRIBUTING.md) - Development guide and contribution guidelines
- [.claude/skills/](/.claude/skills/) - Skill definitions and reference docs

---
> Source: [mcclowes/docusaurus-plugin-glossary](https://github.com/mcclowes/docusaurus-plugin-glossary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
