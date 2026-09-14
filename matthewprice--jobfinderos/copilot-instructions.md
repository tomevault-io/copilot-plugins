## jobfinderos

> Guidance for coding agents and IDE copilots working on this repository (the humans' instructions for the *job-search* agents are in `CLAUDE.md`).

# AGENTS.md — JobFinderOS

Guidance for coding agents and IDE copilots working on this repository (the humans' instructions for the *job-search* agents are in `CLAUDE.md`).

## What this is

An agentic job-search system on Claude Code: three personas in `.claude/agents/`, skills in `.claude/commands/`, doctrine in `config/recruiter_playbook.md`, output in an Obsidian vault at `vault/`. Scheduled runs go launchd → `scripts/scheduler_tick.py` → `scripts/JobFinderOS_run_skill.sh` → `claude -p /skill`.

## Read first

`README.md`, `CLAUDE.md`, `config/recruiter_playbook.md`. When touching automation: `config/scheduler.yaml`, `scripts/`, `vault/Automation/`.

## Rules

1. Nothing about a specific candidate goes into this repo. Personal data lives only in gitignored paths (`config/profile.md` and friends, `vault/` content). If you need an example, use a fictional one.
2. Skills refer to "the candidate" and read every specific (name, employer, comp floor, targets, location) from `config/profile.md`.
3. Never add a code path that sends email or creates Gmail drafts. Never make automation commit or push.
4. Prefer existing entrypoints over new wrappers. One scheduler; no overlapping jobs.
5. Keep skills thin. A skill names its agent in the first line and describes the task; the persona and doctrine live in the agent file.
6. Small, verifiable edits. After an automation change, run `python3 scripts/scheduler_tick.py --dry-run` and `bash scripts/verify_local_automation.sh`.

## Quick commands

```bash
bash scripts/JobFinderOS_check_local_runner.sh
bash scripts/JobFinderOS_run_skill.sh jobs-daily jobs-daily
python3 scripts/scheduler_tick.py --dry-run
bash scripts/JobFinderOS_install_launchd.sh
python3 scripts/jobfinderos_ats_poll.py --dry-run
```

---
> Source: [matthewprice/JobFinderOS](https://github.com/matthewprice/JobFinderOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
