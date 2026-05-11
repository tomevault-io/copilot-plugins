## vitoraos

> This is a personal Obsidian vault organized using the PARA method (Projects, Areas, Resources, Archive). It contains the **Life Operating System (VitoraOS)** — a system for managing all tasks, daily logs, and periodic reviews at this job using an AI coding agent + Obsidian.

# VitoraOS Vault — Copilot Project Instructions

## What This Is
This is a personal Obsidian vault organized using the PARA method (Projects, Areas, Resources, Archive). It contains the **Life Operating System (VitoraOS)** — a system for managing all tasks, daily logs, and periodic reviews at this job using an AI coding agent + Obsidian.

## VitoraOS Overview
VitoraOS lives in `02-Areas/VitoraOS/`. It uses custom prompts and agents (`@log`, `@task`, `@tidy`, `@morning`, `@evening`, `@weekly-review`, `@monthly-review`, `@quarterly-review`, `@annual-review`) to capture and manage work. Task files are individual markdown files with YAML frontmatter that Dataview queries as a database.

### Key Paths
- **Daily Notes:** `02-Areas/VitoraOS/Daily-Notes/YYYY-MM-DD.md`
- **Tasks (active):** `02-Areas/VitoraOS/Tasks/YYYY-MM-DD-brief-slug.md`
- **Tasks (archived):** `02-Areas/VitoraOS/Tasks/Archive/` (done/cancelled — moved here by @tidy)
- **Skills:** `02-Areas/VitoraOS/Skills/` (source of truth for skill behavior)
- **Config:** `02-Areas/VitoraOS/Config/vitoraos-config.md`
- **Decisions:** `02-Areas/VitoraOS/Decisions/` (system design decisions)
- **Dashboard:** `02-Areas/VitoraOS/VitoraOS-Dashboard.md` (Dataview-powered task overview)
- **Weekly Notes:** `02-Areas/VitoraOS/Weekly-Notes/YYYY-WNN.md`
- **Monthly Notes:** `02-Areas/VitoraOS/Monthly-Notes/YYYY-MM-review.md`
- **Quarterly OKRs:** `02-Areas/VitoraOS/Quarterly/YYYY-QN-OKRs.md`
- **Vision:** `02-Areas/VitoraOS/Vision-Roadmaps/3-year-vision.md`
- **Annual Themes:** `02-Areas/VitoraOS/Annual-Themes/YYYY-themes.md`
- **Habits:** `02-Areas/VitoraOS/Habits/` (habit definitions + CSV event logs, checked during @evening)
- **Meetings:** `02-Areas/VitoraOS/Meetings/` (per-meeting notes, archived weekly to `Meetings/Archive/YYYY-WNN/`)
- **Integrations:** `02-Areas/VitoraOS/Config/integrations/` (optional scheduler, calendar, project-workflow configs)
- **Templates:** `99-Templates/` (daily note, weekly review, monthly review, OKR templates)
- **System Design:** `SYSTEM-DESIGN.md` (full system spec)

### Planning Hierarchy
VitoraOS manages execution across a cascading planning hierarchy:
```
3-Year Vision (Vision-Roadmaps/3-year-vision.md)
    ↓
Annual Themes (Annual-Themes/YYYY-themes.md)
    ↓
Quarterly OKRs (Quarterly/YYYY-QN-OKRs.md)
    ↓
Monthly Reviews (Monthly-Notes/YYYY-MM-review.md)
    ↓
Weekly Reviews (Weekly-Notes/YYYY-WNN.md)
    ↓
Daily Notes + Tasks (Daily-Notes/ + Tasks/)
```

Each layer decomposes from the one above. Reviews enforce alignment by pulling context downward. The `@morning` agent proactively prompts the user when reviews are due.

### Task File Schema
```yaml
---
type: task
title: "Human-readable task name starting with a verb"
status: todo | done | backlog | cancelled
priority: P1 | P2 | P3 | P4 | P5
area: [user-defined focus area from Config/vitoraos-config.md]
energy: low | medium | high
depth: deep | shallow (cognitive depth — deep requires sustained creative/analytical cognition; shallow is procedural/interruptible)
duration: 60 (integer minutes)
progress: 0 (optional — minutes completed so far. Remaining = duration - progress)
start: YYYY-MM-DD (earliest schedulable date, optional)
due: YYYY-MM-DD or YYYY-MM-DDTHH:MM (deadline, optional)
deadline-type: hard | soft | none
recurring: true | false (optional — true if generated from Config/recurring-tasks.md)
# --- optional: populated by scheduler integration (see Config/integrations/scheduler.md) ---
scheduler-id: ""              # external scheduler's unique task ID
scheduler-project: ""         # external scheduler's project/workspace
scheduled-start: ""           # when scheduler plans to slot this task
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Priority Rubric
- P1 (Urgent) — Deadline within 2 business days, or actively blocking someone
- P2 (High) — Deadline this week, or important to a stakeholder
- P3 (Medium) — Deadline this month, or important but flexible
- P4 (Low) — Nice to do, no external deadline
- P5 (Backlog) — Ideas to revisit later

### Git Workflow
- **System changes** (skills, config, dashboard, templates) → feature branch + commit
- **Day-to-day usage** (logs, tasks, daily notes) → commit directly to main
- **Always git commit + push to main at the end of @morning and @evening.** Daily note changes, task updates, and context promotions should be persisted to remote immediately so they're backed up and available across machines.

### Review Cadence
The system proactively prompts the user when reviews are due via `@morning`:
- **Daily:** `@morning` (start of day) + `@evening` (end of day)
- **Weekly:** `@weekly-review` (Friday late afternoon, ~30 min)
- **Monthly:** `@monthly-review` (first Monday of month, ~1 hour) — agent-orchestrated
- **Quarterly:** `@quarterly-review` (first week of quarter, ~1-2 hours) — agent-orchestrated
- **Annual:** `@annual-review` (late December / early January, ~2-3 hours) — agent-orchestrated

Monthly, quarterly, and annual reviews are structured conversations where the agent gathers context from lower-level reviews, pre-populates data, and guides the user through each section. The agent drives the review; the user provides reflection and decisions.

### System Rules
- **VitoraOS markdown is the source of truth.** All tasks, logs, and reviews live in markdown.
- Tasks are individual files (not lists) so Dataview can query them and the agent can reason about them.
- Active tasks live in `Tasks/`, completed/cancelled tasks are archived to `Tasks/Archive/` by @tidy. Only scan `Tasks/*.md` during daily operations — never read Archive.
- When creating tasks, always read the `@task` skill file for the full workflow.
- When logging, always read the `@log` skill file for the full workflow.
- Areas of focus are defined in `Config/vitoraos-config.md` and evolve over time. Common patterns: by role function (reporting, data-engineering, admin), by life domain (career, health, family), by project, or a hybrid.
- **Habits** are defined in `Habits/habits.md` and logged to `Habits/habits.csv` during `@evening`. Deep work focus quality is tracked separately in `Habits/deep-work.csv`.
- **Meetings** are noted in `Meetings/` as individual files. `@weekly-review` archives them to `Meetings/Archive/YYYY-WNN/`.
- **Integrations** (scheduler, calendar, project workflow) are optional. See `Config/integrations/README.md`.
- The planning hierarchy flows from 3-Year Vision → Annual Themes → Quarterly OKRs → Monthly/Weekly/Daily. All review skills reference the layers above them.
- `@morning` is responsible for checking review cadence and creating tasks when reviews are due.
- Review skills are agent-orchestrated conversations, not templates for the user to fill out manually.

### Decision Files
Design decisions and significant system changes should be captured as decision files.

- **Location:** `02-Areas/VitoraOS/Decisions/` for VitoraOS decisions. Projects should have their own `Decisions/` folder.
- **Format:** One file per decision. Filename: `YYYY-MM-DD-HHMM-topic-slug.md`
- **Frontmatter:** type, topic, description, scope, date, timestamp, project
- **Body sections:** Context, Decision, Alternatives Considered, Impact
- **When to create:** Any design decision, architecture choice, or significant discussion outcome worth referencing later.

### Context Layer
`02-Areas/` is the context accumulation layer, organized by focus area. Long-lived knowledge (processes, lessons, contacts, decisions) is promoted here from daily logs during `@evening` and `@weekly-review`. `01-Projects/` holds active project documentation and project ideas. The agent should suggest filing reusable insights to `02-Areas/` when it spots them during reviews.

### Relationship to Project Vaults
Software development projects have their own vaults at `~/projects/[project]/docs/` with Agile workflow (epics, stories, sprints). VitoraOS tasks can reference project stories but the two are separate vaults. See `SYSTEM-DESIGN.md` for full details on how they connect.

---
> Source: [verbumeng/vitoraos](https://github.com/verbumeng/vitoraos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
