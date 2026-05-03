## tool-tasks

> tool-tasks — mandatory task tracking via the Rust CLI (vault/Tasks JSONL); no markdown TODOs or parallel trackers.


# Issue tracking with **tool-tasks**

**IMPORTANT:** This project uses the **`tool-tasks`** crate (CLI) for task tracking. Do **not** use markdown TODO lists, `todo_write`, or a second tracker for the same work.

State lives in an append-only **`events.jsonl`** at **`{root}/.tool-llm-git-context/vault/Tasks/events.jsonl`** (Obsidian: **`vault/Tasks/events.jsonl`**). On a **single host**, concurrent **`tool-tasks`** processes coordinate with **`events.jsonl.lock`**. Do not share one store across **machines** without an external protocol.

**`init`** creates the **`Tasks`** directory, a **`.gitignore`** that ignores the lock file, an empty **`events.jsonl`** if missing, and writes **`State.md`** (deterministic projection). Canonical source of truth is **`events.jsonl`**; do not edit **`State.md`** as authority.

### Vault Markdown projection

Every **mutation** that appends to **`events.jsonl`** (for example `task create`, `close`, `dep` changes) regenerates **`State.md`** and per-task notes under **`vault/Tasks/`** automatically. You do **not** need to run **`sync-vault`** after each change for correctness.

Run **`sync-vault`** to rebuild Markdown from the log **without** appending an event—for example after manual repair, copying the store, or if derived files were edited out-of-band.

### Concurrency and parallel agents

**Implementation:** an **OS advisory exclusive lock** on **`events.jsonl.lock`** (via **`fs4`**: `flock`-style on Unix, `LockFile` on Windows) wraps every replay/read and every mutation (including **`State.md`**). **NFS / some network filesystems** may not honor these locks reliably — prefer local disks for the store.

**Safe patterns**

- **Same machine**: Multiple agents may run **`tool-tasks`** concurrently; the lock serializes access to one store.
- **Disjoint stores**: `cargo run -p tool-tasks -- --root /path/to/other-workspace …` so each workspace has its own log (no automatic merge).
- **Multi-host**: Not supported in-process — use disjoint **`--root`**, a merge protocol, or an external queue — see **`history/2026-04-05-tool-tasks-concurrency-parallel-agents.md`**.

## How to run the CLI

From the repo root, always wrap with devenv (see `shell.mdc`):

```bash
devenv shell -- cargo run -p tool-tasks -- <COMMAND>
```

Optional **global** flag (before the subcommand):

- **`--root <PATH>`** — workspace root used to resolve the store (default: current directory).

Use **`--toon`** on read commands when you need machine-parsed output ([TOON](https://github.com/toon-format/toon-rust)).

**Combined context:** when you need git + tasks + LLM session in one picture, run **`git status`** (or similar), **`tool-tasks status --toon`**, and read **`.tool-llm-git-context/current-session.json`** (and optionally **`tool-agent-forum`** commands) yourself. Task **mutations** still go only through **`tool-tasks`**.

Full usage: `devenv shell -- cargo run -p tool-tasks -- --help` and `… <subcommand> --help`.

## Commands (accurate to v1)

| Area | Command | Notes |
|------|---------|--------|
| Bootstrap | `init` | Creates **`vault/Tasks/`**, **`.gitignore`**, empty **`events.jsonl`** if missing, **`State.md`**. |
| Projection | `sync-vault` | Explicit rebuild of **`State.md`** and bucket notes from **`events.jsonl`** (no new event). Mutations already refresh these when they append. |
| Summary | `status [--toon]` | Prints counts: `closed`, `blocked`, `in_progress`, `ready`. **`ready`** = non-closed, `open`, not dependency-blocked (see read model). |
| Tree | `tree [--toon]` | Parent/child forest; non-closed only. Terse text or TOON. |
| List | `list [--toon] [--id …] [--title-contains …] [--status …] [--parent …] [--claimed-by …]` | Default list excludes closed unless `--status closed`. |
| Clean | `clean [--all] [--force]` | Without `--all`: append `task_removed` for closed tasks. `--all --force` truncates the entire event log (destructive). |
| Create | `task create --title <TITLE>` | Prints new task **id** on stdout. |
| Rename | `task set-title --id <ID> --title <TITLE>` | |
| Close | `task close --id <ID>` | |
| Claim | `task claim --id <ID> --by <LABEL>` | Sets `claimed` + `claimed_by`. |
| Unclaim | `task unclaim --id <ID>` | |
| Set status | `task set-status --id <ID> --status <STATUS>` | `open`, `closed`, `blocked`, or `claimed`. |
| Parent | `task set-parent --id <ID> --parent <PARENT_OR_none>` | Use literal `none` to clear parent. |
| Blockers | `task dep add --blocked <ID> --blocker <ID>` | `blocked` cannot proceed until `blocker` is **closed**. |
| Unblock | `task dep remove --blocked <ID> --blocker <ID>` | |

There is **no** separate `ready` subcommand: use **`status --toon`** for the `ready` count, then **`list --toon`** / **`tree --toon`** to inspect tasks. **`list` rows do not embed blocker edges**; an `open` task may still be blocked by `task dep` until its blockers are closed—trust the **`ready`** count in **`status`** for “how many are actually pick-up-able.”

## Agent workflow (minimal)

1. **Ensure store:** `devenv shell -- cargo run -p tool-tasks -- init`
2. **Session context:** `devenv shell -- cargo run -p tool-tasks -- status --toon`
3. **Pick work:** `… list --toon` / `… tree --toon`; prefer tasks with `status: open` when `ready > 0`
4. **Claim:** `… task claim --id <ID> --by <agent-or-user>`. **Also claim every ancestor** (walk **`parent_id`** to the root epic) with the **same `--by`**, so the whole spine is **`claimed`**. Stored task **`status`** is only `open`, `claimed`, `blocked`, or `closed`—there is **no** `in_progress` status to set; **`status --toon`** may still print an **`in_progress`** *count* (read model), which is different.
5. **Discover follow-ups:** `… task create --title "…"` then optional `… task set-parent` / `… task dep add`
6. **Done:** `… task close --id <ID>`

## vs. older docs

Historical notes may mention **bd (beads)** or **`.tool-tasks/`**. For this repository, **tool-tasks is authoritative** for task CRUD and blocker edges; the store path is **`vault/Tasks/`** under **`.tool-llm-git-context/`**.

## Anti-patterns

- **Claiming a leaf or child without claiming parents** — always **`task claim`** the **root→…→your task** chain with one **`--by`** label.
- **Hand-editing** **`events.jsonl`** under **`.tool-llm-git-context/vault/Tasks/`** or **`.tool-agent-forum/events.jsonl`**: forbidden — use the CLIs only; see **`no-direct-events-jsonl-edits.mdc`**.
- Markdown TODOs or `todo_write` for **project** task state (use **tool-tasks**).
- Relying on **NFS or flaky network FS** for the store when you need strict single-writer semantics (locks may be weak).
- Expecting issue **types**, **priorities**, **labels**, or **swarm** commands—v1 is intentionally smaller; encode intent in **title** or **parent**/**dep** structure if needed.

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
