## claude-vigil

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vigil is a CLI that runs `claude -p` unattended against a queue of tasks. Each task gets its own git worktree on a `vigil/<id>-<slug>` branch (slug derived from `task.title` or the prompt; falls back to plain `vigil/<id>` when neither is meaningful). The source repo is never modified directly. When a task finishes the runner auto-commits anything Claude left uncommitted and pushes the branch to `origin` if it's configured. Quota handling is purely reactive: a task only pauses when Anthropic flips a limiter into a pausing status (`"blocked"` — window crossed — or `"rejected"` — this specific request was just refused with a 429; `PAUSE_STATUSES` in `models.py` is the canonical set), and resume is scheduled at the pausing limiter's `resets_at + 60s` buffer.

## Commands

```bash
uv sync --extra dev          # install with test deps
uv run pytest -q             # full test suite
uv run pytest tests/test_queue.py::test_next_runnable_respects_dependencies   # single test
uv run ruff check .          # lint
uv run mypy src/             # type-check (strict)

# CLI (from an editable install)
uv tool install --editable .
vigil doctor                 # pre-flight checks
vigil calibrate              # one-shot probe of `claude -p`
vigil add '<prompt>' --model sonnet --scope-write 'src/auth/**' --budget-cost 1.5
vigil start                  # foreground; SIGINT or `vigil stop` exits cleanly
```

`README.md` and `README.zh.md` are the user-facing docs and they document every flag and design choice in detail — read them before redesigning behavior.

## State layout (split intentionally)

Two roots, owned by `vigil/config.py`:

- **`VIGIL_HOME`** = `platformdirs.user_data_dir("vigil")` (e.g. `~/Library/Application Support/vigil/` on macOS) — internal state: `vigil.db`, `reports/`, `STOP`, `calibration.json`, `config.json`.
- **`VIGIL_USER_HOME`** = `~/.vigil/` — developer-facing artifacts: `worktrees/<id>/` and `logs/`. Short, no-spaces path so `cd` and `tail` are ergonomic on every OS.

Both are overridable via env vars of the same names — required for isolated demos / tests (running `vigil clear` against your real queue is otherwise unrecoverable). The override is module-level, so it must be set on the process that imports `vigil.config`.

Anything you put under platformdirs is hidden; anything under `~/.vigil/` is meant to be opened by the developer. Don't move worktrees or logs back into platformdirs.

## Architecture: how a task flows

1. **`vigil/cli.py`** is the only entry point (Click group). It calls `ensure_dirs()` + `configure_logging()` before any subcommand and resolves the active language via `i18n.resolve_lang` (CLI flag > `VIGIL_LANG` > config file > `$LANG` > default).
2. **`vigil/queue.py`** is the SQLite source of truth. All status transitions (`pending → running → done/failed/needs_review/...`) go through `TaskQueue.update`. `next_runnable()` is DAG-aware (a task is only runnable when every `depends_on` is `DONE`). All datetimes in the DB are naive UTC — `_to_naive_utc()` normalizes anything tz-aware before insert.
3. **`vigil/runner.py`** owns the per-task lifecycle. `main_loop` installs SIGINT/SIGTERM handlers, runs crash recovery (any task left `RUNNING`/`WRAPPING_UP` from a killed daemon is marked `INTERRUPTED`), and alternates between `runner.run(pending)` and `runner.resume(paused)`.
4. **`vigil/worktree.py`** creates `~/.vigil/worktrees/<id>/` on a fresh `vigil/<id>-<slug>` branch from a **locked base SHA**. `derive_branch_name(task.id, task.title or task.prompt)` builds the branch (`slugify` keeps ASCII alnum + CJK + `-_`, collapses runs of `-`, truncates to 40 chars; empty slugs fall back to `vigil/<id>`). The stored `task.branch` is authoritative — the legacy `branch_name(task_id)` helper survives only as a fallback for pre-rename rows. Where the base SHA comes from depends on `vigil add` flags:
   - default (no flags) → `resolve_current_head(cwd)` captures the source repo's current HEAD as a SHA at add time
   - `--base <ref>` → `resolve_base(cwd, ref)` resolves the given branch/tag/SHA at add time
   - `--depends-on <id>` → `task.base_sha` left NULL at add time; the runner resolves the dep's branch tip via `_dep_branch_name(dep_id)` (reads `dep.branch` from the DB, falls back to `vigil/<dep_id>` for legacy rows) right before creating the worktree, then persists it back
   Critical: `resolve_*` always returns a **SHA**, not a symbolic ref — `HEAD` inside a worktree advances as the agent commits, so we have to capture the starting commit at creation time for `diff_stats` to be accurate. `--base` and `--depends-on` are mutually exclusive at the CLI level.
5. **`vigil/claude.py`** spawns `claude -p` with `--print --output-format stream-json --verbose --permission-mode bypassPermissions --disallowed-tools AskUserQuestion --allow-dangerously-skip-permissions`. `StreamSession.events()` yields parsed JSON dicts plus two synthetic events: `_parse_error` and `_exit`.
6. The runner's stream loop (`_spawn_and_stream`) feeds each event through:
   - **`vigil/quota.py`** (`QuotaGuard`) — observes `rate_limit_event` snapshots via `vigil/usage.py` (`UsageTracker`). Triggers a pause iff some limiter's status lands in `PAUSE_STATUSES` (`{"blocked", "rejected"}`), regardless of type — once the server says stop, the next request fails. Anthropic uses both values in the wild: `blocked` when the window has crossed the wall, `rejected` on a 429 reply for a request that just got refused. There is no soft 90%/95% pre-emptive threshold: an earlier design used one as a wrap-up budget buffer, but the guard wasn't re-seeded mid-task so it never fired in practice, and reacting only to server-side signal keeps the rule honest. **`IGNORE_FOR_PAUSE`** (`overage`/`weekly`/`monthly` + variants) is now a *display* filter — `binding()` and the report renderer use it to suppress billing markers from the user-facing diagnostic view, but `should_pause()` and `reset_at()` always honor pausing statuses on any type.
   - **Token budget** (soft, `task.budget.max_tokens`) — accumulated per assistant message, marks `needs_review` once exceeded but lets the current event finish.
   - **Cost budget** (hard, `task.budget.max_cost_usd`) — passed straight to `claude --max-budget-usd`; Claude itself terminates the call.
   - **Wall-clock budget** (`task.budget.max_minutes`) — checked between events.
   - **Shutdown** (`runner._shutdown_requested` from signal handler / `STOP_FILE`) — also checked between events.

## End-of-task checkpoint: auto-commit + push

When the stream loop exits in a non-paused state, `runner._checkpoint_and_push(task, outcome)` runs:

1. **`worktree.auto_commit(task_id, message)`** — `git add -A && git commit` inside the worktree. No-op when there's nothing pending (Claude already committed, or made zero changes). Before staging, `_prune_meta_files` deletes leftover `VIGIL_SUMMARY*.md` from older runs so resumed tasks don't sweep up the previous run's untracked drop. Message comes from `_build_checkpoint_message(task, session_id=outcome.session_id)` and follows Conventional Commits per `.claude/commands/git-commit.md`:
   - **Primary (Claude-generated):** resume the just-finished session and ask `commit_message.generate` for `{title, type, scope}`. Cheap because the prompt cache is hot. When the user didn't supply `--title`, the returned `title` is written back to `task.title` in SQLite so the dashboard and `vigil show` stop displaying the raw slash-command prefix.
   - **Fallback (local heuristic in `_heuristic_checkpoint_header`):** kicks in when there's no `session_id`, when `claude` isn't on PATH, or when the AI call errors / returns malformed JSON. `infer_commit_type_and_scope(worktree, prompt, title)` picks type from changed-file shape first (`tests/**` → `test`, etc.) and falls back to keyword matching with path tokens stripped (so `api-spec.md` no longer trips the `spec` keyword). Subject comes from `task.title` if set, otherwise `_extract_subject` — first prose line of the prompt, skipping `/zcf-workflow ...` slash commands and `<system-reminder>` blocks, with a last-resort `<cmd> <basename>` synthesis when nothing prose-shaped survives.
   - **Trailer:** ` [vigil <id>]` always appended; header capped at 72 chars including trailer.

2. **`worktree.push_to_remote(task_id, branch, source_repo)`** — `git push -u origin <branch>`. Silently skipped when `origin` isn't configured (e.g. local-only repos). Network/permission failures are captured into the result dict and logged, never raised — the task report records the outcome in `report.json#push`.

Why both run unconditionally on the success path: downstream tasks fork from a dependency's branch *tip*. If t001 doesn't commit, the tip stays at the base SHA and t002's `--depends-on t001` becomes effectively a no-op (t002 sees none of t001's work). Push isn't required for the local dependency chain but keeps remote state honest and survives a machine loss.

The quota wrap-up path (`_do_wrapup`) runs the same `auto_commit` + `push_to_remote` after `WRAPUP_PROMPT` returns, as a safety net for when Claude ran out of quota mid-`git commit`. The fallback commit message uses `wip: vigil checkpoint for <id> (quota pause)`.

If you change either function's signature or the message format, update `tests/test_worktree_checkpoint.py` — that suite is the only thing that catches drift.

## The two escape-hatch markers

Both are agreed-upon strings the runner scans for in assistant text. They exist because the unattended setup has no human to ask:

- **`DECISION_NEEDED: <one-sentence question>`** — emitted by Claude (per the `RUNTIME_DIRECTIVE` prepended in `vigil/prompts.py`) when it would otherwise need to ask. The runner sets `task.status = NEEDS_REVIEW` and surfaces the question prominently in `vigil show` and the report.
- **`SCOPE_VIOLATION_REQUESTED: <reason>`** — emitted when the scope hook keeps blocking a write Claude legitimately needs. Same handling: `NEEDS_REVIEW` with the reason captured.

If you change either marker, update `_extract_marker_tail` in `runner.py`, the prompt strings, the scope guard's block reason, and the i18n strings (`decision_needed_*`, `scope_escape_*`).

## Pause-on-blocked + reset-time resume

`QuotaGuard.should_pause()` flips when any limiter (5h / weekly / overage / future types) reports a status in `PAUSE_STATUSES` (`"blocked"` or `"rejected"` — see `models.py`). Once True it's sticky for the rest of the guard's lifetime. The runner reacts:

1. The current `claude` subprocess is `terminate()`'d (SIGTERM, then SIGKILL after 30s).
2. A second short `claude -p` call runs `WRAPUP_PROMPT` in the same worktree to try writing `PROGRESS.md` and a `wip:` commit. That call has its own `--max-budget-usd 0.50` cap. It may itself fail when the block is real (no quota left) — that's fine, step 3 catches it.
3. After the wrap-up call returns, the runner unconditionally runs `worktree.auto_commit` (no API call; preserves whatever the agent wrote even when wrap-up couldn't) and `push_to_remote` so the wip state is preserved both locally and on `origin`.
4. The task is set to `PAUSED_QUOTA` with `resume_at = limiter.resets_at + WAKE_BUFFER_SECONDS` (60s by default) — `resets_at` is taken from Anthropic's own data on the pausing limiter, not a guess.
5. `main_loop` sleeps in 5-minute slices until `resume_at` passes, then `runner.resume(task)` re-spawns `claude -p` in the same worktree with `RESUME_PROMPT` and `--resume <session_id>` so Claude's local conversation history is rehydrated and the agent continues from exactly where it stopped — no re-reading files to rebuild context.

The wrap-up is its own `stream_events` call, not a `StreamSession` — it doesn't need to be interruptible.

Why no soft pre-emption: an earlier 90% threshold was intended to keep budget for wrap-up, but the guard wasn't re-fed during the stream loop (the sidecar's refresh is opt-in), so a slow climb through 90% inside a single task was always missed anyway. The pure-reactive design is honest — only the server's pausing signal (`blocked` or `rejected`) causes a pause — and the auto-commit on the paused path ensures work is preserved even when the wrap-up call itself fails for lack of quota.

`_is_transient_failure` (the retry classifier) also short-circuits on the same signal: if any `outcome.rate_limits` entry has a status in `PAUSE_STATUSES`, the failure is treated as a real wall hit and is *not* retried. Without this belt-and-suspenders, an event-ordering corner case could let the `should_pause()` check miss the pause and the runner would then burn 3 retries (30s / 60s / 120s) against a wall that won't move for an hour or more.

## Transient-failure auto-retry

`claude -p` sometimes fails for reasons that aren't the task's fault — stream idle timeouts, network blips, partial responses, occasional non-zero exits. Vigil retries these automatically up to `MAX_RETRIES_PROCESS` (3) times before giving up.

The retry path piggybacks on the same `PAUSED_QUOTA` machinery as the quota-pause flow — same status, same `_resume_due_tasks` poller, same wake_at — distinguished by the `pause_reason` prefix:

- `"5h blocked at …"` / `"five_hour rejected at …"` etc. → quota pause → `RESUME_PROMPT`, **same Claude session via `--resume <session_id>`**. The Anthropic API prompt cache is cold after the 5h+ gap and the first message takes a full re-prefill, but that's strictly cheaper than dropping the session and forcing Claude to walk the source tree all over again. Session state lives locally (`~/.claude/projects/.../session.jsonl`) and persists across the gap — `--resume` rehydrates the full conversation history. Fallback to a fresh session when `task.session_id` is null (e.g. claude crashed before its `system.init` event).
- `"retry-after-error: …"` → transient retry → `RETRY_PROMPT`, **same Claude session via `--resume <session_id>`** (cache is still warm, costs fractions of a cent)

Flow when `_write_report` would have classified status as FAILED:

1. `_is_transient_failure(outcome, rs)` filters: NOT decision/scope-escape (Claude opted out), NOT budget exhaustion, NOT interrupted. Everything else (`is_error=true`, non-zero exit, parse error) counts.
2. If transient AND `task.retry_count < MAX_RETRIES_PROCESS`: `_schedule_retry` re-queues as `PAUSED_QUOTA`, `retry_count += 1`, `resume_at = now + RETRY_BACKOFF_SECONDS[retry_count - 1]` (30s / 60s / 120s).
3. A retry report lands at `report.json` with `retry_scheduled` + cumulative `retries[]` list, so consecutive failures build up the history rather than overwriting.
4. `main_loop` wakes at `resume_at`, calls `runner.resume(task)`. `resume()` inspects `pause_reason`, picks `RETRY_PROMPT` + passes `task.session_id` through `_stream_and_finalize → _spawn_and_stream → build_argv(extra_flags=["--resume", session_id])`.

`task.session_id` is populated in the stream loop the first time a `system.init` event lands, and persists across retries. If something kills the daemon between attempts, the SQLite row survives and the retry resumes correctly on restart.

If the (N+1)-th attempt also fails transiently, `_schedule_retry` returns False (budget exhausted) and the task lands in real FAILED.

## Self-summary (cheap because of cache)

After a task completes (`DONE` or `NEEDS_REVIEW` only — never `FAILED`/`PAUSED_QUOTA`/`INTERRUPTED`), `vigil/summary.py` re-spawns `claude` with `--resume <session_id>` and sends `SELF_SUMMARY_PROMPT`. Because the session cache is hot, the call costs cents. The output is parsed with progressive fallbacks (`json.loads` → strip code fences → brace-slice) into a `SelfSummary` Pydantic model and lands in `report.json#self_summary`.

If parsing fails, `report.json#self_summary_meta.parse_error` records why — never silently drop the failure.

## Scope hook is a separate process

`vigil/hooks/scope_guard.py` is invoked by **Claude Code itself**, not by Vigil's main process. Wiring:

1. When a task has `scope.write` globs, `_prepare_scope_hook()` writes a per-task `.vigil-settings.json` to the worktree that registers `vigil _scope-hook` as a `PreToolUse` hook for `Edit|Write|MultiEdit|NotebookEdit`.
2. `_scope-hook` is a hidden Click subcommand (in `cli.py`) that just calls `vigil.hooks.scope_guard.main()`.
3. The hook reads the tool payload from stdin, looks up `VIGIL_WRITE_SCOPE` (colon-separated globs) and `VIGIL_WORKTREE_ROOT` from env vars Vigil set on the `claude` subprocess, and either exits silently (allow) or writes `{"decision": "block", "reason": ...}` to stdout.
4. The block surfaces back to Claude as a tool-result message; the runner detects the `"vigil scope violation"` substring in `_extract_block_reason` and records it in `outcome.scope_blocks` for the report.

When debugging scope behavior, remember the hook runs in a **subprocess of `claude`**, not of `vigil`. Its only inputs are stdin and the env vars listed above. Logging to disk from inside the hook is fine (it has the same FS access as `claude`), but it can't import any Vigil-side state.

## i18n: canonical JSON + per-language Markdown

`report.json` is the canonical English-keyed machine record. Both `<id>.md` (English) and `<id>.zh.md` are rendered fresh from it on every report write via `vigil/i18n.py`'s `LOCALES` table. To add a language:

1. Add a key to `SUPPORTED_LANGS` and a sub-dict to `LOCALES` (missing keys fall back to English).
2. Add it to the `_auto_detect()` mapping if relevant.
3. `report.write_summary_md` and `report.write_dashboard` automatically write a file per supported language.

`t()` is process-global and reads from `_current_lang` set once in the CLI entry point. **Don't pass `key` or `lang` as `**fmt` kwargs** to `t()` — they collide with the function signature; use `name`/`value`/`current` instead.

## Subprocess + datetime conventions

- All shelling-out goes through `subprocess.run(..., capture_output=True, check=False)` and reads `returncode` explicitly. We don't raise on non-zero — every git command checks the return code and self-heals where possible (e.g. `worktree.create()` runs `git worktree prune` and deletes dangling branches before retrying).
- Datetimes everywhere are naive UTC. `_utcnow()` in `runner.py` and `_to_naive_utc()` in `queue.py` are the only places this normalization should happen — don't sprinkle `datetime.utcnow()` into new modules without thinking about whether the value will be compared against a tz-aware one.

## Web dashboard (`vigil web`)

`vigil/web.py` is a FastAPI app on top of the same SQLite DB and `reports/*.json` files the CLI uses. `create_app()` is the test entry point (`tests/test_web.py` uses `fastapi.testclient.TestClient`); `serve()` runs uvicorn in the foreground. The frontend is one inline HTML string with vanilla JS, polling `/api/tasks` + `/api/status` every 5 s; markdown rendering is delegated to `marked` from a CDN.

**Write paths** (added after read-only v1):
- `POST /api/tasks` — add a task (mirrors `vigil add`)
- `DELETE /api/tasks/{id}` — single-task remove with `purge_worktree` / `purge_artifacts` / `force` query params. Refuses `running`/`wrapping_up` unless `force=true` (returns 409).
- `GET /api/clear/preview` + `POST /api/clear` — bulk clear with a typed-count safety gate. `confirm_count` must match the live filtered count exactly, else 400.
- `POST /api/daemon/{start,stop}` — spawn/STOP-file the daemon.

There's still no flock between the web app and `vigil start`: the safety comes from refusing to mutate `running`/`wrapping_up` rows, plus the daemon's own crash recovery on next launch. If you add another write endpoint, replicate that pattern.

If you add an endpoint, also add a TestClient test — the test suite is the only thing that catches accidental schema drift between `report.json` and the dashboard's expectations. Screenshot regen for docs is via `playwright` against a sandbox launched with `VIGIL_HOME=/tmp/...` (see commit history).

## Logging

`vigil/logging_setup.py` attaches a `DailySizeRotatingFileHandler` to the root logger writing to `~/.vigil/logs/vigil-YYYY-MM-DD.log` (50 MB cap → numbered backups within the day, new file on date change). It's idempotent and called once from the CLI group. New modules should `log = logging.getLogger(__name__)` and not configure their own handlers.

Per-task event logs (full stream-json) live at `~/.vigil/logs/<task_id>.jsonl` and are appended by the runner. The probe log at `~/.vigil/logs/calibrate.jsonl` is **overwritten** on every `vigil calibrate` (not appended) — useful for diagnosing the most recent probe but don't expect history there.

---
> Source: [jzb1006/claude-vigil](https://github.com/jzb1006/claude-vigil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
