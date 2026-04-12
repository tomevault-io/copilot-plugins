## nexclipper

> This project is a clipper automation tool that uses the YouTube API. All code must be structured, well-maintained, standardized, and designed for easy future development. No simple fixes or lazy troubleshooting — every solution should be thorough and production-quality.

# Project Instructions

## Project Overview
This project is a clipper automation tool that uses the YouTube API. All code must be structured, well-maintained, standardized, and designed for easy future development. No simple fixes or lazy troubleshooting — every solution should be thorough and production-quality.

## User Context
- The user has ADHD — keep everything clean, structured, and organized. No clutter, no loose files, no messy output. Unstructured things cause headaches.

## Project Structure Rules
- Test/utility scripts go in `backend/scripts/`, not in the backend root
- Test output files (`.txt`) must be cleaned up after use — do not leave them scattered
- Keep the repo tidy at all times

## Agent Behaviour

### No Hallucination Policy
Never assume, guess, or argue based on outdated information. If unsure about something:
1. Search for the latest documentation first
2. If still unsure, ask the user — do not insist or argue
3. Trust what the user tells you. If the user says something is correct, accept it and move on
4. Do not waste time debating — verify quickly or defer to the user

### Skills
Before starting any task, check the skills in `.claude/skills/` and apply any relevant skill instructions. Pick only the most relevant skill(s) — don't load all of them.

### Commit & Push Policy
After every change, even small ones, always commit and push immediately. Do not wait for all tasks to be completed. Each commit should represent a single, atomic change — change by change.

### Changelog Logging
After every change, create or update the changelog at `docs/logs/changelogs.md`. Each entry must include at minimum:
- **What** — What was changed
- **Why** — Why the change was made
- **Who** — Who made/requested the change
- **When** — Timestamp of the change
- **Where** — Which part of the project was affected
- **How** — How the change was implemented
- **Files** — Full file paths of all files that were changed

### Planning & Reports
When the user asks for a "plan", the agent must:
1. Consult the skills in `.claude/skills/` to find relevant guidance for the topic
2. Produce a full, detailed plan/report
3. Output the report to `docs/reports/{YYYY-MM-DD}_{title}.md`

### PRDs (Product Requirements Documents)
When the user asks for a "PRD", the agent must:
1. Consult the skills in `.claude/skills/` to find relevant guidance and standards
2. Produce a full, detailed PRD covering goals, scope, requirements, user stories, acceptance criteria, and technical considerations
3. Output the PRD to `docs/prds/{YYYY-MM-DD}_{title}.md`

### Issue Tracking
Any issues — whether mentioned by the user (and not immediately fixed) or discovered by the agent independently — must be documented at `docs/issues/{YYYY-MM-DD}_{issue-name}.md`. Each issue file must include:
- Description of the issue
- Why it matters
- Where it was found (file paths, components)
- Suggested fix or next steps
- Current status

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitdevaldo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gitdevaldo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
