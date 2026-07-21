## multiagent-kit

> > You are developing/maintaining **the multiagent-kit framework itself** (not using it to build some other project).

# CLAUDE.md — multiagent-kit (top-level context for developing this framework)

> You are developing/maintaining **the multiagent-kit framework itself** (not using it to build some other project).
> Read this file first each session; for the fuller backstory, decision rationale, and roadmap see [`docs/DESIGN-AND-HISTORY.md`](docs/DESIGN-AND-HISTORY.md); for the user-facing guide see [`README.md`](README.md).
> For changes that conflict with this file, confirm with the user before acting.
> Language: this repo is English-first (content, prompts, comments). When collaborating with the maintainer, reply in the language they write to you in.

## What this is

A **portable multi-agent collaboration framework**: dropped into any project's `.claude/`, it provides one entry slash command **`/team <task>`**; the current session's Claude acts as the **Lead (orchestrator)** and uses the Agent tool to orchestrate 5 specialist subagents (planner / explorer / developer / tester / reviewer) to iterate automatically around a development task, records the process into an on-disk ledger, and supports long-thread resumption (`/team continue`).

It is **not** a built-in Claude Code feature — it's made purely of this repo's `commands/team.md` (the command playbook) + `agents/*.md` (the subagent definitions).

## Hard constraints (do not violate when making changes)

1. **Project-agnostic (generic)**: the framework content must **never contain** any specific project's business terms, paths, URLs, or secrets. Let it obtain project constraints by reading the target project root's `CLAUDE.md`/`AGENTS.md` at runtime, rather than hardcoding some project's specifics into it.
2. **No heavy dependencies, works offline**: only Markdown + optional shell; don't introduce runtimes/packages/binaries that need installing.
3. **Cross-OS**: wherever commands are involved, give both PowerShell (Windows) and Bash (*nix).
4. **Only stable, documented frontmatter fields** — agents: `name`/`description`/`tools` (optional `model`); command: `description`/`argument-hint`/`allowed-tools`. Don't bet on experimental/version-dependent fields; if you must use one, mark it clearly and keep a graceful fallback.
5. **Lead-exclusive ledger writes**: only the Lead writes `.claude/team-state/`; subagents are read-only and return summaries — to avoid concurrent write conflicts.
6. **Subagents don't include the `Agent` tool**: only the Lead can orchestrate, preventing infinite self-forking.
7. **No auto-commit/deploy by default**: a run only produces changes; whether to publish is left to the user or the target project's convention.

## Directory layout

```
multiagent-kit/
├─ CLAUDE.md                  # this file (context for developing the framework)
├─ README.md                  # user-facing: install/usage/mechanism
├─ LICENSE                    # MIT
├─ docs/
│  └─ DESIGN-AND-HISTORY.md   # full design notes + development history + roadmap
├─ .claude-plugin/plugin.json # optional: plugin install metadata
├─ agents/                    # the 5 subagent definitions → go into .claude/agents/ on install
│  ├─ planner.md  explorer.md  developer.md  tester.md  reviewer.md
├─ commands/
│  └─ team.md                 # entry command /team = the Lead's orchestration playbook
└─ templates/                 # ledger reference templates (the command has the same built in; optional to copy)
   └─ MISSION.md  BACKLOG.md  JOURNAL.md  DECISIONS.md
```

## How to verify when changing this framework

Because the deliverable is "Markdown that drives Claude's behavior", verification relies on:
1. **Structure/format**: all files present, frontmatter YAML valid and using only stable fields.
2. **Genericity**: grep the whole repo to confirm no traces of any specific project (business terms/URLs/secrets).
3. **End-to-end smoke test**: install `agents/` + `commands/` into a real project's `.claude/`, **restart the session**, run `/team ...` on a tiny task, and confirm the Lead creates the ledger → delegates explorer/planner → runs the developer/tester/reviewer loop → wraps up; then `/team continue` to confirm it resumes from the ledger.
   - Note: new commands/subagents are **only scanned at session startup**, so after changes you must open a new session for them to take effect.

## Collaboration conventions

- Sketch the approach before making changes; for structural changes (role responsibilities / orchestration flow / command arguments), confirm with the user first.
- Don't add heavy dependencies or change the established design structure on your own.
- Every time you add/change a feature, update `README.md` (user's view) and `docs/DESIGN-AND-HISTORY.md` (tick off / add roadmap items) as you go.

---
> Source: [sbnlight/multiagent-kit](https://github.com/sbnlight/multiagent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
