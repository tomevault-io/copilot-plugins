## augmented-lean-delivery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a collection of specialized Claude Code skills for Lean Software Development and XP coaching. These skills are designed to be self-contained, portable, and help developers deliver value in the smallest, safest, most reversible steps.

## Architecture

### Skill Structure

Each skill follows a standardized two-file structure:

```
skill-name/
├── SKILL.md       # Core instructions loaded first (YAML frontmatter + core process)
└── REFERENCE.md   # Detailed reference material (loaded on demand)
```

**Progressive disclosure pattern**: Claude loads `SKILL.md` first for core instructions. `REFERENCE.md` is only loaded when detailed reference information is needed, keeping token usage efficient.

### YAML Frontmatter Format

Each `SKILL.md` begins with YAML frontmatter:

```yaml
---
name: skill-name
description: Brief description for skill activation triggers
allowed-tools:
  - Read
  - AskUserQuestion
---
```

- **name**: Must match the directory name
- **description**: Critical for automatic skill activation. Include key trigger words and use cases
- **allowed-tools**: Restricts which tools the skill can access

### Current Skills

1. **hamburger-method** (175 lines) - Applies Gojko Adzic's Hamburger Method for vertical story slicing
2. **story-splitting** (245 lines) - Detects oversized stories and applies splitting heuristics
3. **complexity-review** (474 lines) - Reviews technical decisions against 30 complexity dimensions
4. **micro-steps-coach** (613 lines) - Breaks work into 1-3 hour steps with expand-contract pattern

## Development Guidelines

### When Creating/Modifying Skills

1. **Keep SKILL.md focused**: Core process, mandatory steps, examples. Aim for under 500 lines.

2. **Use REFERENCE.md for details**: Extended examples, advanced patterns, comprehensive lists, detailed heuristics.

3. **Description is critical**: The `description` field controls when skills auto-activate. Include specific trigger words and scenarios.

4. **Progressive quality gradients**: When generating options, use consistent quality scales:
   - Manual/hardcoded → Semi-automated → Fully automated → Scalable → Enterprise-grade

5. **Mandatory sections in SKILL.md**:
   - Core Process (numbered steps)
   - Clear examples showing input → output
   - When to use this skill

### File Naming Conventions

- Skill directories: lowercase with hyphens (`hamburger-method`, not `HamburgerMethod`)
- Core file: `SKILL.md` (uppercase)
- Reference file: `REFERENCE.md` (uppercase)

### Installation Paths

Skills can be installed in two locations:

- **Personal**: `~/.claude/skills/skill-name/` (available across all projects)
- **Project-specific**: `.claude/skills/skill-name/` (committed with the repo)

This repository stores skills at the root level for development, not in `.claude/skills/`.

## Testing Skills

### Recognition Test
```
Ask Claude: "What skills do you have available?"
```

### Functional Tests

Test each skill with its trigger scenarios from README.md:

- **hamburger-method**: "I need to implement email notifications. How do I slice it?"
- **story-splitting**: "Story: As admin, I want to manage users and assign roles"
- **complexity-review**: "We need Kafka, event sourcing, CQRS, and auto-scaling"
- **micro-steps-coach**: "I need to rename the database column 'email' to 'email_address'"

Expected behaviors are documented in README.md lines 203-234.

## Philosophy & Principles

Skills embody these Lean/XP principles:

- **Small, safe steps**: Risk grows faster than the size of the change
- **Zero downtime**: All changes preserve current functionality
- **Reversibility**: Favor reversible decisions
- **YAGNI**: Build the simplest system that works today
- **Basal cost**: Every line of code has ongoing maintenance cost

## Attribution

Skills package knowledge from:

- **Eduardo Ferro** - Lean Software Development, complexity dimensions, basal cost (https://www.eferro.net/)
- **Gojko Adzic** - Hamburger Method (https://gojko.net/)
- **Various XP practitioners** - Story splitting heuristics

Always maintain attribution when modifying skills.

---
> Source: [eferro/augmented-lean-delivery](https://github.com/eferro/augmented-lean-delivery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
