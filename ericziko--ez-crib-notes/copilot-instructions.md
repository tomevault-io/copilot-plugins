## ez-crib-notes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Directory-Specific Rules
- **DO NOT TRAVERSE OUTSIDE THE CURRENT DIRECTORY** 
    - You can read ANY files beneath the current directory
    - You are NOT to leave this current directory

## Repository Overview

**ez-crib-notes** is a personal knowledge base and technical documentation repository containing notes on .NET development, PowerShell, SQL Server, architecture patterns, and software engineering practices. Notes are organized chronologically by date and tagged with topics for searchability.

## Repository Structure

- **Date-organized folders** (`YYYY-MM-DD/`): Notes grouped by the date they were captured
- **Root-level files**: Standalone documents and tutorials (e.g., `PowerShell Background Processes Tutorial.md`, `README.md`)
- **.git/**: Git version control tracking all note changes
- **.gitignore**: Tracks only `.DS_Store` (macOS metadata)

## Note Format & Metadata

Notes follow a markdown format with YAML frontmatter containing:

```yaml
---
uid: <uuid>  # or title/date/tags depending on note
title: <Title>
created: <ISO8601>
modified: <ISO8601>
tags:  # Optional
  - tag1
  - tag2
---
```

Both UUID-based and title-based metadata formats are used across the repo. When editing notes, preserve the existing frontmatter format and update the `modified` timestamp.

## Common Development Tasks

### Viewing Note Structure
```bash
ls -la /Users/ericziko/gitHub/ericziko/ez-crib-notes  # List dated folders
git log --oneline -n 10  # View recent commits
```

### Adding or Editing Notes
- New notes: Always create in appropriate date folder (or root if cross-cutting topic)
- Include frontmatter with metadata (title, date, tags, uid)
- Prefer kebab cased file names
- Commit with `git add <file>` and `git commit -m "<note title>"`
- Use descriptive commit messages matching note titles (see git history for style)

### Searching Notes
Use grep to find topics across the knowledge base:
```bash
grep -r "CQRS" --include="*.md"
grep -r "PowerShell" --include="*.md"
```

### Viewing Git History
```bash
git log --oneline  # All commits
git log -- <file>  # History of specific note
git diff <commit>  # Changes in a commit
```

## Key Technical Topics Covered

The repository contains deep dives on:
- **.NET & C#**: CQRS pattern, Clean Architecture, Dependency Injection, HttpClientFactory, Error Handling, Health Checks, Observability
- **Database**: SQL Server metadata reflection, SqlLocalDB for unit testing, integration tests
- **PowerShell**: Modules, CLI scripting, background processes, VIM PSReadLine
- **Patterns**: Architecture testing, Mediator pattern, plugin systems, SRE practices (toil)
- **DevOps & Monitoring**: OpenTelemetry, background jobs (Hangfire/Temporal), diagnostics tools

## Special Note Types

- **🤖❓ Prompts**: Notes titled with "🤖❓" prefix are responses to specific AI/ChatGPT questions about technical topics

- When creating responses 
  - use a pattern of 

  ```markdown

    - # 🤖❓    - to indicate AI answers
    - ## 🤖💡   - to indicate AI answers
    - ## 🤖💬❓ - to indicate the AI querying me for further information
   - Please use emoji with headings to indicate different types of subject matter
  ```


- **Tutorial Notes**: Standalone guides (e.g., `PowerShell Background Processes Tutorial.md`)
- **WF Notes**: Workflow-specific notes (e.g., `WF Notes for SqlLocalDB XUnit Testing/`)

## Git Workflow Notes

- Commit messages often use shorthand: `cp` = "copy", `cm` = some standard meaning
  - These short comments are because I am lazy - please make better commit messages for me
- Merge commits indicate collaborative pulls from GitHub
- Linear history preferred for readability of knowledge evolution

## When Working in This Repo

1. **Before editing**: Check the note's existing frontmatter format and preserve it
2. **Adding new notes**: Decide between date-organized folder (topical) or root level (cross-cutting)
3. **Committing**: Use clear, descriptive commit messages (see history for examples)
4. **Searching**: Use grep with `--include="*.md"` to find related notes
5. **No build/lint/test cycle**: This is a documentation-only repository—no compilation needed

## Environment & Tools

The development environment has the following tools installed and available:

- **ripgrep (rg)**: Fast recursive search tool
- **fd**: Fast alternative to `find` for file discovery
- **bat**: Syntax-highlighted cat replacement

Claude Code will prefer dedicated tools (Glob, Grep, Read) when available, but these tools can be used directly in bash commands when needed.

---
> Source: [ericziko/ez-crib-notes](https://github.com/ericziko/ez-crib-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
