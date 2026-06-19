## claude-blog-multilingual

> This repository contains **Claude Blog Multilingual**, an extension for

# Claude Blog Multilingual — International Blog Publishing Extension

## Project Overview

This repository contains **Claude Blog Multilingual**, an extension for
[Claude Blog](https://github.com/AgriciDaniel/claude-blog) that adds
multilingual content creation, translation, cultural adaptation, and
international SEO capabilities.

**4 sub-skills, 1 specialized agent, and 2 reference files** enable complete
multilingual blog publishing from a single command.

## Architecture

```
claude-blog-multilingual/
  CLAUDE.md                                     # Project instructions (this file)
  skills/
    blog-translate/                             # SEO-optimized translation
      SKILL.md
      references/
        translation-rules.md                    # Format tables, quality criteria
        cultural-adaptation.md                  # Cultural profiles per locale
    blog-localize/                              # Cultural deep-adaptation
      SKILL.md
    blog-multilingual/                          # Orchestrator (write + translate + SEO)
      SKILL.md
    blog-locale-audit/                          # Multilingual quality control
      SKILL.md
  agents/
    blog-translator.md                          # Specialized translation subagent
  install.sh                                    # Unix/macOS installer
```

## Commands

| Command | Purpose |
|---------|---------|
| `/blog translate <file> --to de,fr,es` | Translate a post into target languages |
| `/blog localize <file> --locale de-DE` | Culturally adapt a translated post |
| `/blog multilingual <topic> --languages de,fr,es` | One-command: write + translate + localize + hreflang |
| `/blog locale-audit <directory>` | Audit multilingual content for completeness and SEO parity |

## Integration

This extension **requires** [Claude Blog](https://github.com/AgriciDaniel/claude-blog)
to be installed. It extends `claude-blog` with multilingual capabilities and
integrates `seo-hreflang` validation logic from
[Claude SEO](https://github.com/AgriciDaniel/claude-seo).

```
claude-blog (required)         This Extension
┌───────────────────┐     ┌──────────────────────┐
│ blog-write        │◄────│ blog-multilingual    │
│ blog-analyze      │◄────│   (Orchestrator)     │
│ blog-schema       │◄────│                      │
│ blog-seo-check    │◄────│ blog-translate       │
└───────────────────┘     │ blog-localize        │
                          │ blog-locale-audit    │
claude-seo (optional)     │                      │
┌───────────────────┐     │ blog-translator      │
│ seo-hreflang      │◄────│   (Agent)            │
└───────────────────┘     └──────────────────────┘
```

## Development Rules

- Keep SKILL.md files under 500 lines / 5000 tokens
- Reference files should be focused and under 200 lines
- Follow kebab-case naming for all skill directories
- Agents invoked via Task tool with `context: fork`, never via Bash
- ISO 639-1 for language codes, ISO 3166-1 Alpha-2 for regions

---
> Source: [Chriss54/claude-blog-multilingual](https://github.com/Chriss54/claude-blog-multilingual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
