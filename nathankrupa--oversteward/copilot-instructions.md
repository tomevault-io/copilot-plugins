## oversteward

> <!-- [oversteward:managed | synced: 2026-03-09] -->

<!-- [oversteward:managed | synced: 2026-03-09] -->
@~/.claude/shared/souls/chestertron.md

At session start, check `~/.claude/shared/inbox.md` for updates. If entries exist, review them and apply any relevant changes, then clear the file.
<!-- [oversteward:managed:end] -->

---

# Working Guidelines

Core rules: `~/.claude/CLAUDE.md`

---

# Project-Specific Configuration

<!-- [oversteward:local] -->

## Python Environment

**Use the project venv** (uv-managed) for running Python commands:
```bash
uv run python <script>
uv run python -m <module>
uv run pytest
```

`uv` auto-creates `.venv/` from `pyproject.toml` on first invocation; subsequent `uv run` calls implicitly activate it. No need to `source .venv/bin/activate` manually.

Bootstrap on a fresh checkout:
```bash
uv sync --extra dev
```

## Session worktree discipline

**One git worktree per session.** Parallel Claude/human sessions that share the
primary checkout collide — a `git checkout`/`switch` in one strands another's
uncommitted work. Start every unit of work in its own worktree:

```bash
scripts/dev/new-session.sh <name>   # → .claude/worktrees/<name> on session/<name>
```

The `.claude/hooks/guard_main_worktree.py` `PreToolUse(Bash)` hook **refuses
branch checkout/switch in the primary worktree** (file restores, `git worktree
add`, and linked worktrees are exempt). Deliberate one-offs (a promote, a
rebase) prefix the command with `CLAUDE_ALLOW_MAIN_GIT=1`.

This is the estate-wide standard, canonical here in `shared/scripts/dev/` and
deployed to every repo's `.claude/hooks/` + `scripts/dev/`. See OVERSTEWARD.md
§ "Session-per-worktree discipline" for the rollout + new-project bootstrap.

## Running a command with `.env` loaded (sanctioned `with_test_env` runner)

When a command needs secrets from `.env` (the dockerized test backend, `make
verify`, integration `pytest`), **do not** `source .env` or put a `DB-URL=...`
assignment on the command line. The credential-hygiene hook blocks those shapes
because an unquoted `&` in a connection string (every Neon URL carries
`&channel_binding=require`) leaks through bash job control onto stderr — and
stderr is captured in the transcript. Do **not** hand-roll a one-off in-process
shim either; use the sanctioned runner:

```bash
scripts/dev/with_test_env.py make verify
scripts/dev/with_test_env.py -- pytest -k integration
scripts/dev/with_test_env.py --env-file .env.test python -m mytool
```

It parses `.env` in-process (never through the shell), merges the values into the
environment **without ever printing a value**, and `exec`s the target command so
it inherits them. Existing process environment wins over `.env` (matching
`load_dotenv`'s default). Stdlib-only and dependency-free, so it deploys to every
repo. It is a canonical byte-copy in the `shared/scripts/dev/` family (same
deploy + drift-tracking as `new-session.sh` and `guard_main_worktree.py`) — see
`@~/.claude/shared/references/credential-hygiene.md` for the rule it satisfies.

## Architecture

Principles: `@~/.claude/shared/references/architecture-principles.md`

**Layer map for this project:**
- **OUTER:** `scripts/` (coordinator, gather, diff, sow, sweep), `.claude/skills/`
- **MIDDLE:** Orchestration logic within scripts (Phase 2 — currently stubs). As scripts grow beyond stubs, extract services to `src/oversteward/`
- **INNER:** `registry.yaml` (config), `contexts/` (per-repo instructions), `shared/` (canonical source files)

Oversteward is currently a config-management project in Phase 1 (manual sync). When Phase 2 scripts are implemented, business logic (diff analysis, conflict resolution, sow decisions) belongs in a middle-layer service, not in the scripts themselves.

## Key Components

- **registry.yaml** — manifest of all managed contexts (soul, personas, sync behavior)
- **shared/** — canonical soul and persona source files (deploy to `~/.claude/shared/`)
- **contexts/** — per-context local override instructions (one file per managed repo)
- **scripts/** — coordinator, gather, diff, sow, sweep (Phase 2)
- **.claude/skills/create-persona.md** — scaffold and deploy new personas

## Key Files

- **Architecture state:** `architecture.md` — machine-readable snapshot of repos, cross-repo seams, load-bearing invariants, known liabilities, recent moves. **Read at scope/plan time** when a task may touch more than one repo or any §3 invariant. If a row is wrong, fix it before continuing. Dispatch agents do NOT read this — it's a scoping-time tool. **Before scoping "edit X and propagate through Y" work, verify the propagation surface Y actually exists** — `git grep` for the managed-block markers / §2 seam and confirm the consumer reads it; if it's absent, building it is in-scope, not a follow-up (see architecture.md §6).
- **Spec:** `OVERSTEWARD.md` (architecture and all design decisions)
- **Ledger:** `Stewards_Ledger.md` (project status and session log)
- **Todo:** `MASTER_TODO.md` (active) + `TODO_BACKLOG.md` (queued) + `TODO_COMPLETED.md` (archive)
- **Session:** `SESSION_STATE.md` (local-only "where we left off" scratchpad — **gitignored, never committed**; see § Session handoff)

## Session handoff

`SESSION_STATE.md` is a **per-machine local scratchpad** — gitignored, never
committed. The durable, shareable handoff lives in two already-synced places:
**auto-loaded memory** (`~/.claude/projects/.../memory/`) and **GitHub issues**
(the real work tracker). This is deliberate: the resident Telegraph operator runs
in the primary checkout and can't cleanly branch per handoff, so committing
`SESSION_STATE.md` directly to `master` used to leave local-only commits that
never pushed *and* a `master` that fell behind origin — drift that compounded
every session (OS#90).

Drift-prevention rules for the primary checkout:
- **Never commit `SESSION_STATE.md`** (it's gitignored) — write handoff facts to
  memory + issues instead.
- **Never commit anything directly to the primary checkout's `master`** — all
  changes flow through a worktree → PR (see PR workflow in `~/.claude/CLAUDE.md`).
- **At session start, while the tree is quiet:** `git pull --ff-only origin master`
  to pick up merged PRs. Because the primary checkout never carries local
  commits, this fast-forward can never diverge or fail. Do **not** pull
  mid-session (worktree-isolation), and you do **not** need to pull on every
  merge — read current code mid-session from origin (`git show
  origin/master:<path>`) rather than mutating the live tree.

## Tool Registry

**When looking for a CLI tool, script, or entry point — read `data/tool_registry.md` first.**

Regenerate after adding/removing tools:
```bash
uv run python scripts/tools/generate_tool_registry.py
```

## Sync Instructions (Phase 1 — Manual)

When Nathan asks for a sync check:
1. Read `registry.yaml` for the current context list
2. Deploy `shared/` to **both** Claude home directories:
   - Windows: `C:\Users\natha\.claude\shared\`
   - WSL2: `/home/natha/.claude/shared/` (via `\\wsl.localhost\Ubuntu-24.04\home\natha\.claude\shared\` or `wsl --exec bash -c "cp ..."`)
   Both must stay in lockstep — repos on either filesystem resolve `@~/.claude/shared/...` against their own home. Skipping the WSL mirror silently breaks AG/GS (and any future WSL repo).
3. For each context (skip `skip_sow: true`): check whether the managed block matches what registry specifies
4. Generate a report in `reports/YYYY-MM-DD.md`
5. Present proposed changes; wait for approval before running sow.py

<!-- [oversteward:local:end] -->

---
> Source: [NathanKrupa/OverSteward](https://github.com/NathanKrupa/OverSteward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
