## ai-rules-bank

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI rules bank repository that stores reusable AI assistant guidelines following the Cline rules format (https://docs.cline.bot/features/cline-rules). The repository is designed to create shareable rule sets that can be referenced across different projects.

## Architecture

The repository uses a memory linking system:

- `CLAUDE.local.md` - Local project memory that references generic rules via `@` links
- `generic/` - Contains reusable rule modules that can be shared across projects
- Rule files use `@filename.md` syntax to reference other rule files within the same directory
- Rules are organized by topic (e.g., `testing.md` for testing guidelines)

## Memory System

The memory system works through file references:
- `@generic/filename.md` - References files in the generic directory from the root
- `@filename.md` - References files in the same directory
- Rules can be chained: A → B → C where each file references the next

When working with memory files, always preserve the linking structure rather than flattening unless explicitly requested.

## File Standards

- All markdown files should end with a newline character
- Follow the existing format for rule documentation with clear section headers
- Use descriptive titles that match the content purpose

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hota911)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hota911)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
