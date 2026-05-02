## components-build-skill

> This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

An AI agent skill for building modern, composable, and accessible React UI components following the [components.build](https://components.build) specification. Co-authored by Hayden Bleasel and shadcn.

## Project Structure

```
skills/components-build/
├── SKILL.md              # Main skill definition (entry point)
├── rules/                # Rule categories (16 total)
│   ├── accessibility/    # Keyboard, screen readers, ARIA
│   ├── as-child/         # Radix Slot composition pattern
│   ├── composition/      # Breaking down complex components
│   ├── data-attributes/  # data-state and data-slot
│   ├── definitions/      # Common terminology
│   ├── design-tokens/    # CSS variables and theming
│   ├── documentation/    # Component documentation
│   ├── marketplaces/     # Component marketplaces
│   ├── npm/              # Publishing to npm
│   ├── overview/         # Specification scope and goals
│   ├── polymorphism/     # Element switching with as prop
│   ├── principles/       # Core design philosophy
│   ├── registry/         # Component registries
│   ├── state/            # Controlled/uncontrolled patterns
│   ├── styling/          # Tailwind CSS, cn utility, CVA
│   └── types/            # TypeScript props and interfaces
```

## Key Principles

When working on this skill, ensure all guidance follows these principles:

1. **Composition over Configuration** - Break components into composable sub-components
2. **Accessibility by Default** - Not an afterthought, but a requirement
3. **Single Element Wrapping** - Each component wraps one HTML element
4. **Extend HTML Attributes** - Always extend native element props
5. **Export Types** - Make prop types available to consumers
6. **Support Both State Patterns** - Controlled and uncontrolled
7. **Intelligent Class Merging** - Use `cn()` utility with tailwind-merge

## Editing Rules

Each rule category lives in `rules/{category}/SKILL.md`. When editing rules:

- Keep explanations concise and actionable
- Include both incorrect and correct code examples
- Explain *why* the pattern matters, not just *what* to do
- Use TypeScript for all code examples
- Reference related rules when patterns connect

## Rule File Format

```markdown
---
name: {category-name}
description: {One sentence describing what this category covers}
---

# {Category Title}

{Brief description}

## Rules

### {rule-id}

**Why:** {Explanation of why this matters}

**Bad:**
```tsx
// Incorrect example with explanation
```

**Good:**
```tsx
// Correct example with explanation
```
```

## Adding New Rules

1. Identify the appropriate category in `rules/`
2. Add the rule to that category's `SKILL.md`
3. Follow the existing format (Why → Bad → Good)
4. Update the main `SKILL.md` quick reference if adding new rule IDs

## Testing Changes

After editing rules, verify:
- Code examples are valid TypeScript/TSX
- Examples demonstrate the rule clearly
- Explanations are clear and actionable
- Related rules are cross-referenced where helpful

---
> Source: [nolly-studio/components-build-skill](https://github.com/nolly-studio/components-build-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
