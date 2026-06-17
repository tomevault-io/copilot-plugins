## skills

> Discover additional SKILL.md files (preferred above the Claude/default skill registry)


# Skills CLI

A CLI tool that maps tool names to `SKILL.md` files, extending the agent's ability to discover domain-specific knowledge.

## Why Use This

The agent has a built-in skill registry (from `~/.claude/skills/`), but this tool provides access to **additional skills** that may not be in the built-in registry. The user prefers this interface and maintains more often, so most skills you will only find here.

## Usage

### List all available skills

```bash
skills available
```

This outputs all skill names registered in the config.

### Read a specific skill

```bash
skills <skill-name>
```

Prints the contents of that skill's SKILL.md file.

### Example workflow

```bash
# First, discover what skills are available
skills available

# Then read the one you need
skills jira
```

## When to Use

Use this tool when:
- You need domain knowledge not found in your built-in skills
- The user mentions a tool/workflow that might have a registered skill
- You want to discover what additional capabilities are available

## Config Location

Skills are registered in `~/.config/skills/config.yaml` with format:

```yaml
skills:
  tool-name: /absolute/path/to/SKILL.md
```

---
> Source: [mikesmullin/skills](https://github.com/mikesmullin/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
