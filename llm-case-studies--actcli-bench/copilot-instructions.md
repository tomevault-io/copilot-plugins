## actcli-bench

> Welcome. If you are a fresh Claude session opening this repo, read this

# Claude session orientation — ActCLI-Bench

Welcome. If you are a fresh Claude session opening this repo, read this
file first.

## What this repo is

`ActCLI-Bench` is part of the ActCLI group: terminal-native AI tooling
(see `README.md`). Recent work has been on the bench emulator, cursor
positioning correctness, and instrumentation refactor — see
`docs/INSTRUMENTATION_REFACTOR.md` and `docs/POSTMORTEM_CURSOR_PTY.md`
for the most current technical context.

## How work moves through this repo

This repo follows a small **sprint-pack process** lifted from
`llm-case-studies/panel-runner` and `llm-case-studies/office-clerk`.
The process is documented at:

- `docs/process/README.md` — overview and pointers
- `docs/process/framework.md` — the framework manual (rules, branch
  naming, sprint shape, who-does-what)
- `docs/process/templates/sprint-pack/` — paste-ready templates for a
  new sprint
- `docs/process/templates/testing/` — paired testing-request templates
- `docs/process/example.md` — pointer to a real completed sprint in
  `panel-runner` for shape reference

**Three roles:**

- **Orchestrator** (typically a Claude session running where the human
  drives) — designs sprints, writes briefs, runs the git ritual.
- **Implementer** (typically a coding agent on a coding host, e.g.
  DeepSeek via OpenCode) — executes inside the brief's fence.
- **Validator** (typically a tester on a separate host) — independently
  reruns probes, captures evidence, returns a verdict.

A new Claude session arriving cold should NOT start coding before
reading the framework manual and checking
`docs/expert-briefs/initiatives/*/active/` for any in-flight sprint.

## Initiative and sprint folders

When the first sprint pack is created in this repo, expect:

```
docs/expert-briefs/
  README.md, INDEX.md, LESSONS.md
  initiatives/<initiative>/
    README.md, INDEX.md
    active/<YYYY-MM-DD_slug>/      ← in-flight sprints
    completed/<YYYY-MM>/<slug>/    ← finished sprints + merge notes

testing/initiatives/<initiative>/<slug>/
  request.md, result.md, evidence/
```

Until that scaffolding lands, treat existing `docs/*.md` files as the
source of truth for technical context, and `docs/process/` as the
source of truth for HOW work should be organized going forward.

## Cross-session memory note

This repo's `~/.claude/projects/-...-ActCLI-Bench/memory/` (the
machine-local Claude memory directory) is not portable across hosts or
sessions. Anything that future sessions need to know belongs in the
repo — under `docs/process/`, in sprint-pack briefs, or in
`docs/expert-briefs/LESSONS.md` once that file exists.

## ActCLI group context

Sister repos in `llm-case-studies`:

- `ActCLI` — the actuarial CLI itself
- `ActCLI-TE` — terminal engine (MIT-licensed core)
- `ActCLI-Bench` — this repo, "Inter-AI facilitated sessions"
- `ActCLI-Round-Table` — multi-agent debate format
- `ActCLI-Extensions` — Chrome bridge and test rigs
- `ActCLI-HIC` — hardware insight console
- `ActCLI-Round-Table-V1` — private spike

The sprint-pack process can scale across these repos when shared
concerns (diagnostics interfaces, integration validation) need
coordinated work. Cross-repo sprints belong in a shared coordination
repo or are mirrored — see `docs/process/framework.md` for guidance.

## Sprint readiness checklist (for the orchestrator)

Before handing a sprint to an implementer:

1. Have you read the most recent technical postmortems in `docs/`?
2. Have you checked `docs/expert-briefs/initiatives/*/active/` for any
   in-flight work that conflicts?
3. Does the brief explicitly name the implementation host, validation
   host, working branch, merge target, and validation branch?
4. Is the testing request paste-ready (host safety, exact commands,
   evidence file paths)?
5. Are out-of-scope items called out as explicitly as in-scope?

If any of these is missing, the brief is not yet handoff-ready.

---
> Source: [llm-case-studies/ActCLI-Bench](https://github.com/llm-case-studies/ActCLI-Bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
