## mind-skills

> A collection of Claude Code skills for fetching and displaying information from external sources. Designed for `npx skills add` installation.

# Daily News Skills - Project Memory

## Project Overview

A collection of Claude Code skills for fetching and displaying information from external sources. Designed for `npx skills add` installation.

## Skills

| Skill | Purpose | Data Source |
|-------|---------|-------------|
| `ai-daily` | AI industry news aggregation | RSS (smol.ai) |
| `fs-street` | Farnam Street blog articles | RSS (fs.blog) |
| `trending-skills` | Skills.sh ranking tracker | Playwright |
| `github-topics` | GitHub topic trending repositories | GitHub API |

## Directory Structure

```
skills/
├── ai-daily/
│   ├── SKILL.md
│   ├── scripts/
│   └── references/
├── trending-skills/
│   ├── SKILL.md
│   └── src/
└── github-topics/
    ├── SKILL.md
    └── src/
```

---

## Architecture Philosophy

**Stateless Skills**: Each skill is a pure query tool following Unix philosophy:
- Fetch data from external source
- Return formatted results
- No storage, no state, no historical analysis

**Data persistence** (if needed) is handled by the AI assistant's scheduled task layer, not by individual skills.

---

## Pre-Commit Workflow

When user says "提交代码" or "commit", execute this workflow BEFORE pushing to remote:

### Step 1: SKILL.md Specification Check

For each `skills/*/SKILL.md`, verify:

**YAML Frontmatter**:
- [ ] `name`: lowercase, numbers, hyphens only (max 64 chars)
- [ ] `name`: no consecutive hyphens (`--`), no leading/trailing hyphens
- [ ] `name`: no reserved words ("anthropic", "claude")
- [ ] `description`: non-empty, max 1024 chars
- [ ] `description`: third-person voice ("Fetches...", "Tracks...", NOT "I can..." or "You can...")
- [ ] `description`: includes WHAT it does + WHEN to use it

**Content Quality**:
- [ ] Total lines < 500
- [ ] No time-sensitive information (use `<details>` for deprecated content)
- [ ] Consistent terminology (don't mix "API endpoint" with "URL", "route", "path")
- [ ] References only one level deep (no nested references)
- [ ] Progressive disclosure pattern used

**Intent Recognition**:
- [ ] Quick Start section with common examples
- [ ] Query Types table with trigger patterns
- [ ] Workflow checklist for complex tasks

### Step 2: Code-Documentation Consistency Check

Verify alignment between code and documentation:

- [ ] CLI commands in SKILL.md match actual script paths
- [ ] Environment variables documented match those in code
- [ ] Categories/classifications match between code and docs
- [ ] Output format examples reflect actual code behavior

### Step 3: Formatting and Grammar Check

- [ ] Markdown syntax is valid
- [ ] Tables are properly formatted
- [ ] Code blocks have language specifiers
- [ ] No broken links or references
- [ ] Consistent heading hierarchy
- [ ] No trailing whitespace in code blocks

### Step 4: User Experience Check

- [ ] Clear and concise language
- [ ] Actionable examples (not abstract)
- [ ] Error messages are helpful
- [ ] Troubleshooting section covers common issues
- [ ] Low barrier to entry for new users

### Step 5: Update CHANGELOG.md

Record changes in `CHANGELOG.md` following format:

```markdown
## [version] - YYYY-MM-DD

### Added
- New features

### Changed
- Modifications to existing features

### Fixed
- Bug fixes

### Removed
- Removed features
```

---

## Specification Reference

Based on `/root/skill-rules/SKILL-RULE.md`:

### Name Field Rules
- Max 64 characters
- Only lowercase letters, numbers, hyphens
- No XML tags
- No reserved words: "anthropic", "claude"
- No consecutive hyphens
- No leading/trailing hyphens

### Description Field Rules
- Non-empty, max 1024 characters
- No XML tags
- MUST use third-person voice
- Include: what it does + when to use it

### Good Description Examples
```
✅ "Processes Excel files and generates reports. Use when working with spreadsheets."
✅ "Extracts text from PDF files. Use when user mentions PDFs or document extraction."
❌ "I can help you process Excel files"
❌ "You can use this to process files"
```

### Checklist for High-Quality Skills

**Core Quality**:
- Description is specific with key terms
- Third-person voice throughout
- SKILL.md < 500 lines
- Consistent terminology
- One-level references only

**Code and Scripts**:
- Scripts handle errors gracefully
- No magic numbers (all values documented)
- Dependencies are listed
- Unix-style paths (not Windows)
- Validation steps for critical operations

---

## Common Patterns

### Workflow Pattern
```markdown
## Workflow

- [ ] Step 1: Parse input
- [ ] Step 2: Fetch data
- [ ] Step 3: Process results
- [ ] Step 4: Generate output
```

### Query Types Table
```markdown
## Query Types

| Type | Examples | Description |
|------|----------|-------------|
| Rankings | `今天排行榜` `Top 10` | Show rankings |
| Detail | `xxx是什么` | Show details |
```

### Progressive Disclosure
```markdown
## Quick Start
[Basic usage here]

## Advanced Features
See [reference.md](reference.md) for details
```

---

## Version History

See `CHANGELOG.md` for detailed version history.

---
> Source: [geekjourneyx/mind-skills](https://github.com/geekjourneyx/mind-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
