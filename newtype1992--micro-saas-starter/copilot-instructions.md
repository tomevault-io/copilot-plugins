## micro-saas-starter

> This project is designed to be developed with an AI coding agent (Codex CLI).

# Agent Instructions

This project is designed to be developed with an AI coding agent (Codex CLI).

Before making changes, read CONTEXT.md.

## Development Rules

1. Never modify database schema directly.
   Use migrations.

2. Do not introduce new environment variables unless necessary.

3. Always explain architectural changes before implementing them.

4. After making changes:
   - summarize files modified
   - list any manual setup steps required

5. Prefer simple solutions over complex abstractions.

6. Validate backend changes with Supabase before continuing work.

## Safety Rules

- Never delete existing migrations
- Never expose service role keys
- Never assume production credentials

## Output Format

After completing a task, respond with:

1. Summary of work completed
2. Files modified
3. Remaining manual setup steps
4. Recommended next step

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/newtype1992)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/newtype1992)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
