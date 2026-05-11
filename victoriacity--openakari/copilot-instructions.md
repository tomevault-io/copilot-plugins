## claude-skills

> Bridge Claude skills into Cursor — instructs agent to find and use skills from .claude/skills/. See AGENTS.md for unified entry point.


# Claude Skills Bridge

> **Note:** This repo uses `AGENTS.md` as a unified entry point for all AI coding agents (Claude Code, Cursor, opencode). The fallback chain is: Claude Code → Cursor → opencode. Read `AGENTS.md` for an overview, then `CLAUDE.md` for comprehensive instructions.

This repo has skills defined in `.claude/skills/`. Each skill is a directory containing a `SKILL.md` file with instructions for specific workflows (e.g., `/orient`, `/diagnose`, `/compound`).

## How to Use Skills in Cursor

Since Cursor does not have a native Skill tool, use the Read tool to load skill files when needed:

1. **When the user invokes a skill by name** (e.g., `/orient`, `/diagnose`, `/compound`), read the corresponding skill file at `.claude/skills/<skill-name>/SKILL.md` and follow its instructions.

2. **Before starting non-trivial work**, check if a relevant skill exists. Available skills:

   | Skill | When to use |
   |-------|-------------|
   | `orient` | Start of every work session — assess state and select task |
   | `self-audit` | Convention compliance check |
   | `coordinator` | Multi-step coordination |
   | `compound` | End of session — embed learnings |
   | `diagnose` | Something went wrong with data/results |
   | `slack-diagnosis` | Slack-reported issue triage |
   | `postmortem` | Agent reasoning failure analysis |
   | `design` | Designing a new experiment |
   | `develop` | Software development workflow |
   | `architecture` | Infrastructure health check |
   | `critique` | Correctness review of a plan/design |
   | `simplify` | Complexity reduction check |
   | `synthesize` | Cross-session findings synthesis |
   | `gravity` | Recurring manual pattern absorption |
   | `lit-review` | Literature search |
   | `review` | Validate experiment metrics and findings |
   | `audit-references` | Verify citations |
   | `report` | Generate a report |
   | `publish` | Paper submission preparation |
   | `horizon-scan` | Scan for new developments |
   | `feedback` | Process PI/human feedback |
   | `project` | Create new projects — propose (gap analysis) or scaffold (interview + setup) |
   | `refresh-skills` | Audit skills against current code |

3. **To load a skill**: `Read .claude/skills/<skill-name>/SKILL.md` — then follow the instructions in the file exactly.

4. **Skill files may reference additional files** in their directory (e.g., examples, templates). Read those as needed.

## Priority

- Process skills first (orient, diagnose, debugging) — they determine HOW to approach the task.
- Implementation skills second (develop, architecture) — they guide execution.
- When the user says a slash-command like `/orient`, always load and follow the skill.

---
> Source: [victoriacity/openakari](https://github.com/victoriacity/openakari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
