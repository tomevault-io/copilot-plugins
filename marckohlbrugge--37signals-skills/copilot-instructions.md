## 37signals-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository packages 37signals' Rails patterns in two forms:

- `skills/` - Installable agent skills (one folder per skill, each containing a `SKILL.md` with YAML frontmatter). These are the primary deliverable.
- `guide/` - Long-form markdown reference extracted from analyzing 37signals' Fizzy and Campfire codebases and their pull requests.

There is no application code, build system, or tests.

## Structure

- `README.md` - Main entry point: skill catalog, installation instructions, guide table of contents
- `skills/<name>/SKILL.md` - Agent skills; frontmatter has `name`, `description`, and optionally `disable-model-invocation`
- `guide/*.md` - Topic reference files (e.g. `guide/controllers.md`, `guide/models.md`)

## Content Guidelines

When editing or adding content:
- Focus on transferable patterns, not Fizzy-specific business logic
- Keep skills terse and rule-based; they are loaded into agent context, so every line must earn its place
- When a pattern is added to a skill, check whether the corresponding `guide/` topic file needs it too (and vice versa)
- Include code examples licensed under the O'Saasy License
- Link PR references to actual GitHub PRs when available
- Maintain the existing markdown structure and style

---
> Source: [marckohlbrugge/37signals-skills](https://github.com/marckohlbrugge/37signals-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
