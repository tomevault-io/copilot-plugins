## r5-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Radius5/Copainter team internal **Claude Code plugin marketplace**. It is a pure Markdown/JSON repository with no application code, build system, or tests. Each top-level directory is a self-contained Claude Code plugin.

## Plugin Structure Convention

Every plugin follows this layout:

```
<plugin-name>/
  .claude-plugin/plugin.json        # Plugin manifest (name, description, version, author)
  skills/<skill-name>/
    SKILL.md                         # Skill definition with YAML frontmatter trigger phrases
    references/*.md                  # Supporting reference docs the skill reads at runtime
```

- `plugin.json` registers the plugin with Claude Code's plugin loader.
- `SKILL.md` frontmatter `description` field controls auto-invocation (matched against user phrases in both English and Japanese).
- `references/` files are read by the skill during execution, not loaded upfront.

## Current Plugins

- **pr-review** — Structured PR review workflow using `gh` CLI. Copainter-specific (Next.js + Firebase + Stripe). Implements incremental file review, security scanning, and posts reviews via `gh pr review` with AI fix prompts.

## Key Conventions

- All content is in Markdown and JSON — no transpilation or compilation.
- Skills encode anti-hallucination guardrails (e.g., never fetch full diff if >300 lines, never review lock files line-by-line).
- Review output uses `gh pr review` with structured templates including collapsible AI Fix Prompt sections.
- Documentation and skill descriptions are bilingual (Japanese/English).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/radius5-study) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
