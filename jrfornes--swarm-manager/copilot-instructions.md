## swarm-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A file-based orchestrator for running multiple Claude Code agents in parallel,
each in an isolated git worktree, with a static HTML dashboard. **Roadmap:**
evolve into a local **forge host** (many work-items, Attention, bounded
pipeline concurrency) — see `plan.md` Part III. Deliberately dependency-free:
Python 3 stdlib only, no framework, no database, no build step.

## Commands

Start the dashboard server (serves the UI and the JSON/log API on port 8787):

```bash
python3 server.py
```

Launch an agent job (run from the repo root; background with `&` to launch several at once):

```bash
python3 launch_agent.py <role> <repo_path> "<task description>" [--permission-mode MODE] [--timeout SECONDS] [--item NAME] [--skills-dir PATH]
```

`<role>` (`builder`/`tester`/`reviewer`/anything) is just a label shown on the dashboard — roles are not enforced. To actually constrain what an agent may do, pass `--permission-mode` (one of `default`, `acceptEdits`, `bypassPermissions`, `plan`; default `plan` for the `reviewer` role, `acceptEdits` otherwise) or add flags to the `claude` invocation in `launch_agent.py`. `--skills-dir` (default `.claude/skills`) is only recorded in `meta.json` and used by `pipeline.py` to build stage prompts.

`launch_agent.py` exits `0` **only** when the job lands in `done` — `blocked`, `failed`, and timeouts all exit `1`. Chaining, `pipeline.py`'s stage gate, and the future queue worker all depend on that; preserve it if you touch the exit path.

Run one work-item through ordered skill stages (background with `&` to run pipelines in parallel):

```bash
python3 pipeline.py <repo_path> "<work-item description>" --stages spec-author,verifier,implementer
python3 pipeline.py --target my-nx-app "<work-item description>"
python3 pipeline.py --target my-nx-app "<work-item description>" --workflow docs/reference/swarm-workflows/workflow-4-pack-smoke.json --dry-run
```

Stages default to the target repo's workflow manifest (see `docs/workspace-contract.md`). `--dry-run` prints the stage commands without launching anything. `pipeline.py` does **not** validate a manifest stage's `permission_mode`; a typo only surfaces when `launch_agent.py` rejects it at that stage.

Clean up after jobs (worktrees, `agent/*` branches, and `jobs/` entries are never auto-removed). `cleanup.py` handles all three and refuses to discard uncommitted or unmerged agent work unless `--force`:

```bash
python3 cleanup.py <job-id> [<job-id> ...]   # specific jobs
python3 cleanup.py --done                    # every done/blocked/failed job
python3 cleanup.py --all                     # every job that isn't running
python3 cleanup.py --dry-run --all           # preview, change nothing
python3 cleanup.py <job-id> --force          # discard uncommitted/unmerged work too
```

There are no tests, linter, or dependencies to install. "Verify" in this repo means a scripted smoke test: create a throwaway git repo, run a real `launch_agent.py` job (or `pipeline.py --dry-run`) against it, and check the resulting `meta.json` / dashboard state. `plan.md` records a concrete **Verify** step under every phase that has landed, plus a "Testing note" and a retained regression recipe ("Verify workspace abstraction") near the end — reuse those rather than inventing new ones.

## Layout

This repo is the orchestrator; agents never run *in* it. `<repo_path>` must be a separate, real git repo, since `launch_agent.py` runs `git worktree add` inside it. Alternatively, `pipeline.py --target NAME` resolves `repo` from a local `targets.local.json` (template: `targets.example.json`).

There is **no `.gitignore` yet**. `jobs/`, `worktrees/`, and `targets.local.json` (which holds absolute local paths) are runtime/local state and must not be committed; add a `.gitignore` before committing after a real run.

- `target_config.py` — loads `targets.local.json` and each consumer's `.swarm/profile.json` (`skills_dir`, `workflow` paths).
- `docs/workspace-contract.md` — the consumer-repo contract: registry keys, `.swarm/profile.json`, `.swarm/workflow.json`.
- `docs/reference/swarm-workflows/` — reference workflow manifests (1- through 6-stage packs) with a picker README; copy into a consumer's `.swarm/` or point `--workflow` at one.
- `jobs/`, `worktrees/` — per-job state and checkouts, created at runtime. Both are cleaned only by `cleanup.py`. `launch_agent.py` also appends a JSONL debug trace to `jobs/debug-3b000d.log`; it is best-effort, never affects status, and is skipped by `server.py` / `cleanup.py` because it is not a directory.
- `public/` — the static dir `server.py` serves. **`public/index.html` is not checked in**; `server.py` creates an empty `public/` on startup, so the dashboard is a 404 until the page is added.
- **Planned (Part III):** `work-items/` (or `forge/items/`) for forge metadata; `worker.py` for queue + max in-flight pipelines.

## Architecture

The `jobs/` directory **is** the entire state store. There is no database and no shared in-memory state between the entry points — they communicate only through files on disk.

- **`launch_agent.py`** — the writer. For each job it (1) mints an 8-char hex `job_id`, (2) `git worktree add`s a new branch `agent/<job_id>` under `worktrees/<job_id>/` so concurrent agents never touch the same files, (3) runs `claude -p "<task>" --output-format json --permission-mode <mode>` with `cwd` set to that worktree, and (4) writes `jobs/<job_id>/meta.json` (status, role, item, branch, worktree, `skills_dir`, `base_sha`/`end_sha`, timestamps, hoisted `total_cost_usd`/`permission_denials`, and the full parsed JSON result) and `jobs/<job_id>/output.log` (raw stdout/stderr from git + claude). `meta.json` is written atomically (temp file + `os.replace`) so the server never reads a half-written file, and the whole run is wrapped in `try/finally` so a job **always** reaches a terminal status. Status transitions: `starting` → `running` → one of `done` (finished and did its work), `blocked` (ran cleanly but was denied permission — `permission_denials` non-empty), or `failed` (git/worktree error, agent error, or timeout). The outcome is derived in `classify()` from the result payload, **not** the `claude` exit code, which is `0` even when the agent was blocked: `failed` if the exit code is non-zero, stdout is not JSON, `is_error` is set, `subtype != "success"`, `terminal_reason == "api_error"`, or the result text starts with `Unknown command:` / contains `Failed to authenticate`; otherwise `blocked` if `permission_denials` is non-empty; otherwise `done`.

- **`pipeline.py`** — the sequencer. Runs one work-item through ordered skill stages by shelling out to `launch_agent.py <skill> <repo> <task> --item <name> --skills-dir <dir>` once per stage and stopping on the first non-zero exit. The entire staging mechanism is that one flag: `--item NAME` puts the job in `worktrees/<NAME>` on branch `agent/<NAME>`, **creating the worktree only if missing**, so every stage of a work-item shares one tree and the files a stage leaves behind _are_ the handoff — there is no message bus, queue, or handoff protocol, by design. Without `--item`, a job keys its worktree on its own id, which is the original fan-out. Each stage stays an ordinary job with its own `jobs/<id>/` entry and dashboard card, linked to its siblings by the `item` field. The default item name is a 24-char slug of the description plus a 4-hex suffix. Stage config resolution, highest precedence first: `--stages` (skill names only) → `--workflow PATH` (repo-relative or absolute) → the target's `workflow` key in `targets.local.json` → `.swarm/profile.json` `workflow` → `.swarm/workflow.json`; `skills_dir` follows the same chain minus the two flags. A manifest must have a non-empty `stages` array where every entry has `skill`; a stage may also set `permission_mode`, `timeout`, and `task` (in which `{description}` / `{DESC}` are replaced with the work-item text). The default stage task tells the agent to read `<skills_dir>/<skill>/SKILL.md` in the worktree.

- **`server.py`** — the reader. Re-reads every `jobs/*/meta.json` fresh on each request; keeps no cache. Binds to `127.0.0.1` (localhost only) and is threaded (`ThreadingTCPServer`) so a slow `git diff` doesn't block the 2s poller. Routes (`<id>` is validated against real job dirs to block path traversal), plus static file serving from `public/`:
  - `GET /jobs` → JSON array of all job metas, newest `started_at` first.
  - `GET /jobs/<id>/log` → last 200 lines of that job's `output.log` as plain text.
  - `GET /jobs/<id>/diff` → what that job changed, computed live from git. Each job records `worktree`, `base_sha` (the tree's HEAD when it started) and `end_sha` (where it left it), so a stage that later stages have committed on top of is diffed over the bounded `base_sha..end_sha` range, while the stage still at the branch tip is diffed against the live worktree — which keeps uncommitted work and untracked files visible for the job that actually produced them. Jobs predating these fields fall back to the merge-base.
  - everything else → static files.

- **`cleanup.py`** — the only thing that removes state. For each selected job it removes the worktree, then the `agent/*` branch, then `jobs/<id>/`. Stages of one work-item share a worktree, so the tree and branch are removed only when the **last** job that owns them is being cleaned; earlier siblings drop just their `jobs/` dir (`worktree_owners()` / `handled`). Safety without `--force` is asymmetric: a dirty worktree (uncommitted or untracked changes) skips the **whole job**, while an unmerged branch is kept but its worktree and `jobs/` dir are still removed. Active (`starting`/`running`) jobs are skipped unless `--force`.

- **Dashboard contract (`public/index.html`, not yet in the repo)** — the intended page polls `/jobs` every 2s and renders jobs into five columns keyed on status (`starting`, `running`, `done`, `blocked`, `failed`), each card colored by status and showing duration, cost, and a ⚠ badge when `permission_denials` is non-empty. Clicking a card opens a panel with **log** and **diff** tabs (`/jobs/<id>/log`, `/jobs/<id>/diff`). Any static asset must live under `public/`, the `PUBLIC_DIR` `server.py` serves from.

## Roadmap — read before proposing changes

Two docs already track the intended direction; check them before designing anything, so you extend the existing plan instead of re-deriving it.

- **`improvements.md`** — backlog (#9 concurrency, **#12 forge host**). Items #1–#8, #10, #11 are done.
- **`plan.md`** — Part I (hardening) done through Phase 3; **Part III (forge host)** is the active product direction. Part II v1 (`pipeline.py`) and II.1b (workspace abstraction) landed; grouping and verify v2 are polish.

Keep both in sync when you land something — their status tables are how the next session knows what is already built.

## Conventions when extending

The README lists what stays minimal (no DB, no websockets, one-shot stages).
**Part III** adds forge features (work-items, worker, board) without abandoning
the file-based model. Do not add tmux/Babashka to the orchestrator without an
ADR in `plan.md`. Two invariants are load-bearing and easy to break by accident: `meta.json` must stay atomically written (a reader can hit it at any moment), and every job must reach a terminal status even when the process dies mid-run.

---
> Source: [jrfornes/swarm-manager](https://github.com/jrfornes/swarm-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
