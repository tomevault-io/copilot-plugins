## scraut

> > This file is read automatically by Claude Code at the start of every session.

# Scraut — Claude Code Context

> This file is read automatically by Claude Code at the start of every session.
> It contains the non-negotiable rules and architecture for Scraut.
> **Always read this file before writing any code.**

---

## What is Scraut

Scraut is a fully automated, text-file-first Scrum system built on GitHub
infrastructure. Human/team source-of-truth data lives in `workspace/`. Scraut
generated output and internal state live in `.scraut/`. GitHub Actions workflows
read those files, call an LLM when needed, and write back summaries and derived views.

**One sentence:** text files in, automation out.

---

## Non-negotiable architecture rules

These rules must never be violated, regardless of what seems convenient.

### 1. Text files are the ONLY source of truth

The GitHub Projects board is a **derived view**. It is regenerated from text
files by `scraut.scrum.visibility.sync_board`. Never write code that reads the
board to make decisions — always read the text files.

Human-authored source files live under `workspace/`. Bot-generated files,
derived state, summaries, reviews, synced code activity, suggestions, insights,
and milestone planning output live under `.scraut/`.

### 2. Never overwrite human-edited files

Use `create_if_not_exists()` from `scraut.platform.utils.file_utils` for any file
a human might write. Use `atomic_write()` only for bot-generated files.
Bot-generated files always start with `<!-- BOT-GENERATED -->`.

### 3. One file per contributor — zero git conflicts

Standup files: `workspace/sprint/NN/standup/YYYY-MM-DD/[login].md`
Retro files: `workspace/sprint/NN/retrospective/[login].md`
One human = one file. Bots write generated output to `.scraut/`.

### 4. Bot commits always include `[skip ci]`

Every `git commit` message from a script or workflow must end with `[skip ci]`
to prevent infinite workflow loops. Without this, a bot commit triggers another
workflow, which triggers another commit, infinitely.

### 5. LLM is called only for language tasks

Pattern detection, velocity calculation, file routing, issue labelling logic —
these are deterministic scripts. LLM is only called when the task genuinely
requires language understanding: summarisation, sentiment, narrative generation,
suggestion drafting, planning session questions.

### 6. Every script accepts `--dry-run` and `--config`

`--dry-run`: print proposed changes, make no writes or API calls
`--config`: path to an alternative `scraut.yml` (default: `./scraut.yml`)

---

## Three file zones

| Zone          | Who writes             | Examples                                                  | Rule                        |
| ------------- | ---------------------- | --------------------------------------------------------- | --------------------------- |
| Human input   | Humans (GitHub pencil) | `workspace/sprint/NN/standup/DATE/login.md`               | `create_if_not_exists` only |
| Agent input   | AI agents              | `workspace/sprint/NN/standup/DATE/agent-*.md`             | Same format as humans       |
| Bot-generated | GitHub Actions         | `.scraut/sprint/NN/standup/summary/`, `.scraut/insights/` | `atomic_write` OK           |

---

## Core utilities (always use these, never reimplement)

```python
from scraut.platform.utils.config import load_config, get_repo_root, get_workspace_root, get_scraut_root, get_current_sprint, get_team_logins, get_sprint_folder, get_sprint_output_folder
from scraut.platform.utils.file_utils import atomic_write, create_if_not_exists, read_file, extract_section
from scraut.platform.utils.date_utils import get_sprint_dates, is_working_day, days_until_sprint_end
from scraut.platform.github.api import get_github_client, get_issues, get_sp_from_issue
from scraut.platform.github.projects import get_project_id, get_project_items, get_field_ids
from scraut.platform.llm.client import complete, complete_json
from scraut.platform.notifications.slack_post import post_to_slack, send_slack_dm
```

---

## Story point labels

Issues use `sp:1`, `sp:2`, `sp:3`, `sp:5`, `sp:8`, `sp:13` labels.
`get_sp_from_issue(issue)` returns the integer value. Returns 0 if no label.

---

## Sprint labels

Each sprint issue has label `sprint-{N:02d}` (e.g. `sprint-01`, `sprint-14`).
Also `in-sprint` while active. `deferred` when rolled over to next sprint.

---

## scraut.yml location

Always at repo root. Loaded via `load_config()` which caches after first call.
`current_sprint` is the authoritative sprint number. Incremented by `close_sprint.py`.

---

## What NOT to do

- Do not add new root folders unless they represent a real top-level responsibility
- Do not use `print()` for errors — use `logging.error()`
- Do not hardcode repo names — always read from config or `GITHUB_REPOSITORY` env var
- Do not make real API calls in tests — use mocks from `test/mocks/`
- Do not commit without `[skip ci]` in bot-generated commit messages
- Do not ask for confirmation on individual files — implement everything in the phase

---
> Source: [aldhosutra/scraut](https://github.com/aldhosutra/scraut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
