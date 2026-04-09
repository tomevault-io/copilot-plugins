## musix-js

> > **Rule**: Do not add anything to CLAUDE.md unless it is necessary.

# CLAUDE.md

> **Rule**: Do not add anything to CLAUDE.md unless it is necessary.

## Project Overview

musix.js is a unified music service adapter library for TypeScript. See [README.md](README.md) for details.

## Language

All documentation, code comments, commit messages, and issues must be written in **English**.

Exception: Specification content (`docs/specs/*.md`) may be written in other languages.

## Directory Structure

```
docs/
  adr/              # Architecture Decision Records
  llms-txt/         # LLM documentation for libraries
  specs/            # Feature specifications (spec-driven development)
    templates/      # Spec templates (FEATURE, ENHANCEMENT, FIX, REFACTOR)
```

## Spec-Driven Development

This repository follows **Spec-Driven Development**.

1. **Write specs before code** - Always create a specification before implementation
2. **Keep change history** - Update the history section when modifying specs
3. **Follow the spec** - Do not implement features not described in the spec
4. **Derive tests from specs** - Test cases should be based on spec requirements

## Development Workflow

```
/spec-new → /design → [ /session-start → /session-end ]* → PR
```

`*` = Repeat until all requirements pass. Each session focuses on **one requirement**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HotariTobu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HotariTobu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
