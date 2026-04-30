## ralph-wiggum-asset-pricing

> The goal of the project is to generate an academic economics or finance paper.

# CLAUDE.md

The goal of the project is to generate an academic economics or finance paper. 

Always read `spec/paper-spec.md` before doing work.

See `README.md` for the full repo structure and how Ralph (a.k.a. RALF, ralph, the ralph loop) works.

Import `.agent-guidelines/README.md` for shared agent coding, git, and interaction rules.

## Prompt Design Guidelines
- Keep prompts general. Do not use examples to patch current holes in the system.

## Key Files
- `spec/paper-spec.md`: paper specification
- `spec/economic-background.md`: shared economic terminology and background conventions for the current paper; update it when changing the paper topic.

## Folder Structure (Quick Reference)

### Research Project
- `paper/` — LaTeX source
- `code/` — R scripts and analysis
- `data/` — datasets
- `dev/` — scratch work, journals

### Specification
- `spec/` — repo intent and source-of-truth specifications

### Tests
- `tests/` — test and referee definitions (referee scripts prefixed `referee-`)
- `test-results/` — latest test and referee outputs


### Ralph (Permanent)
- `ralph/` — loop script, agent wrapper, prompts

### Ralph Garage (Transient)
- `ralph-garage/` — improvement plan, shared page images
- `ralph-garage/page-images/` — uncommitted page images of the paper
- `ralph-garage/history/` — track ralph loop progress here (iteration logs, run summaries)
- `ralph-garage/loop.log` — console output from the ralph loop

### Dev Logs
- `ralph-garage/agent-logs/` — non-canonical agent debug logs

### Disposable Code and Results
Wipe these folders before making disposable code
- `dev/tmp/` — disposable code
- `dev/tmp/results/` — disposable results

## Sync Issues
- The files are synced via Dropbox and may not be available immediately.
- If files are unreadable or corrupt, stop and ask the user rather than trying to work around the issue. The user can trigger a Dropbox sync to fix it.
- If git fails with "Resource deadlock avoided" on a reflog file (e.g. `.git/logs/refs/heads/...`), ask the user to delete that file from the Mac side. It's a stale Dropbox/FUSE lock.

## Worktrees
- Create git worktrees under `/workspace/worktrees/`.

## Coding Guidelines
- Use R for data analysis.
- For the ralph loop, use minimal bash, and lean on Python for most tasks.
- Use kebab-case for file and folder names.
- Top comment in every script: how to run, inputs, outputs.
- Do not create fallback or try-catch statements unless they are strictly necessary.
- if the user says "what do you think?" it means explain and do not modify any files.
- Check `paper/` and `code/` for starter paper and code examples, including `code/query-wrds-example.R` for WRDS access.

## Git Usage
- For commits that update the ralph algorithm, start with "algo update:"
- For commits where the ralph loop updates the paper, start with "rloop" (e.g., `rloop-01: ...`)
- "The beginning of the run" means the commit immediately before the first `rloop` commit on the current branch — not the merge base with main. Use `git log --oneline <branch> | tail` to find it.
- Treat `ralph-garage/` and `test-results/` as transient artifacts rather than durable source files.
- Normal commits must not include files under `ralph-garage/` or `test-results/`.
- If transient artifacts need to be preserved, commit them separately in an artifacts-only commit. Do not mix them with changes in `ralph/`, `tests/`, `spec/`, `paper/`, `code/`, `README.md`, `CLAUDE.md`, `config-ralph.yaml`, or `dev/journal/`.

## Journal
- When requested, take notes in `dev/journal/`
    - Filename: `YYMMDD[T]HH-[note name].md`
    - Include title, date, and time at the top.

## Time Zone
- Use New York time zone for all timestamps.

## Interaction Guidelines
- The user may refer to "Claude" with other names:
    - Claw, clawed
- The user may refer to "Ralph" with other names:
    - Ralf, ralf, ralph
- When discussing math, respond in Markdown-formatted math 
    - Use $x_t = 2$ for in-line math
    - Use $x_t = 2$ for displaymath

---
> Source: [chenandrewy/ralph-wiggum-asset-pricing](https://github.com/chenandrewy/ralph-wiggum-asset-pricing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
