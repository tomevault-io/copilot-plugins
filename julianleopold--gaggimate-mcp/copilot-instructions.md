## gaggimate-mcp

> **This workspace is for `julianleopold/gaggimate-mcp` ONLY.**

# Claude Code Instructions for gaggimate-mcp

## Repository Scope Restriction

**This workspace is for `julianleopold/gaggimate-mcp` ONLY.**

**NEVER** perform any actions on external repositories (e.g. `Matvey-Kuk/gaggimate-mcp`).
Before any GitHub operation, verify the owner is `julianleopold`. If not, STOP and ask for confirmation.

---

## Changelog Management

When committing significant features or changes, **update the Changelog section in README.md**.

Add to changelog: new features, breaking changes, user-facing bug fixes, API changes.
Skip: minor refactoring, code cleanup, test additions, doc typo fixes.

Format: `### YYYY-MM-DD` with bullet points. Newest entries at the TOP.

---

## TODO List Management

A `TODO.md` file exists in the project root to track pending tasks.

1. **At session start**: Check `TODO.md` for incomplete items and ask if the user wants to work on any
2. **When completing a task**: Move to "Completed" section with date
3. **When discovering new work**: Ask if it should be added to TODO

---

## Skill & Knowledge File Versioning

When editing any file under `agent-skills/` or `agent-instructions/`, **always update the version in the YAML frontmatter** of the modified skill file(s). Use `git rev-parse --short HEAD` + today's date.

---

## Claude Desktop Agent Deployment

The agent instructions and skills for Claude Desktop live in this repo and must be manually uploaded to claude.ai.

### Files involved:
- **System instructions**: `agent-instructions/INSTRUCTIONS.md` — copy-paste into the Claude Desktop project
- **Skills** (5 zip files): `diagnose-skill.zip`, `feedback-skill.zip`, `gaggimate-profiles-skill.zip`, `knowledge-lookup-skill.zip`, `new-coffee-skill.zip` — upload to the project's knowledge

### Stamp & deploy workflow:
When the user wants to update Claude Desktop with the latest instructions/skills, run:
```bash
./scripts/stamp-and-zip.sh
```
This script:
1. Stamps the current commit hash + date into INSTRUCTIONS.md and all skill YAML frontmatter
2. Re-zips all 5 skill folders
3. Prints a summary of what to upload

**Do NOT run this script on every commit** — only when the user explicitly wants to deploy to Claude Desktop.

### Version tracking:
- INSTRUCTIONS.md has a version line: `> **Version:** \`<hash>\` | Last updated: YYYY-MM-DD`
- Each skill has a YAML field: `version: <hash> (YYYY-MM-DD)`
- The user can ask the Claude Desktop agent "what version are your skills?" to verify what's deployed

---
> Source: [julianleopold/gaggimate-mcp](https://github.com/julianleopold/gaggimate-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
