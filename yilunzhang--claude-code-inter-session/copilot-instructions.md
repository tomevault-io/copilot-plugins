## claude-code-inter-session

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An agent-to-agent messaging bus for Claude Code: multiple CC sessions on
the same Unix machine connect to a localhost WebSocket server and
exchange messages that drive actions in the receiving session.

Two install modes, **both supported and tested**:

- **Plugin** (recommended): `/plugin marketplace add …` → `/plugin
  install inter-session@inter-session`, or `claude --plugin-dir <repo>`
  for local dev. Adds `userConfig` (port, idle-shutdown) and
  `monitors.json`. User invokes as `/inter-session:inter-session …`.
- **Standalone skill**: clone or symlink `skills/inter-session/` to
  `~/.claude/skills/inter-session/` (e.g. `ln -s
  <repo>/skills/inter-session ~/.claude/skills/inter-session`). The
  skill is self-contained — `bin/`, `requirements.txt`, and `SKILL.md`
  all live inside `skills/inter-session/`, so a copy or symlink of just
  that subdirectory is a fully working skill. User invokes as
  `/inter-session …` (no plugin namespace). No `userConfig`; override
  defaults via `INTER_SESSION_PORT` / `INTER_SESSION_IDLE_MINUTES` env
  vars if needed.

The skill content (`skills/inter-session/SKILL.md`) is install-mode
agnostic: the connect step has **no upfront dedup check**. It picks a
name and calls `Monitor()` directly. If a monitor is already running
for this CC session, `bin/client.py`'s ppid-flock catches the duplicate
and the new spawn exits with `[inter-session] another monitor for this
session is already running`, which the LLM surfaces via the Error
notifications path. Skipping the pre-check optimizes the common case
(not connected yet → straight spawn, ~50-100ms faster) and lets the
flock be the single source of truth for race-safety. Don't add a
`list.py --self` or `TaskList()` pre-check back into the connect step
— that was tried and reverted because the optimization paid more in
the common case than it saved in the edge case.

Layout follows the conventional `skills/<name>/SKILL.md` auto-discovery
pattern that the current CC plugin schema requires (`"skills": ["./"]`
is rejected with "Path escapes plugin directory"). **`bin/` lives
inside the skill dir** (`skills/inter-session/bin/`). The plugin's
monitor `when` defaults to `on-skill-invoke:inter-session` (lazy);
`bin/auto_start.py` flips it to `always` when the user runs
`/inter-session auto-start on`. Empirically `on-skill-invoke` may not
reliably auto-spawn a working monitor in current CC versions, so the
LLM's `Monitor()` call in the skill is what actually establishes the
connection most of the time.

When CLAUDE.md and other docs reference `bin/<script>.py` as an
abbreviated label, the actual path is
`skills/inter-session/bin/<script>.py`.

Single user, single machine. Unix-only (macOS / Linux / WSL2).

## Common commands

Local dev runs entirely in a project-local venv at `.venv`. The
Makefile bootstraps it on first use (uv preferred, stdlib `venv` as
fallback). System Python is never touched.

```bash
make test                                    # full suite (~49 s, 197 tests)
make test-fast                               # skip subprocess-spawning tests
make clean                                   # remove .venv
```

To run pytest with non-make flags, use the venv's pytest directly:

```bash
.venv/bin/pytest tests/test_server.py -v     # one file
.venv/bin/pytest -k "election" -v            # by substring
```

No build step, no linter configured. Runtime deps live at
`skills/inter-session/requirements.txt` (websockets + psutil); dev
deps inherit those plus pytest via `requirements-dev.txt`. Both reqs
files install into `.venv` via `make test` — there's nothing to install
by hand.

## Architecture (big picture)

Three process classes share a localhost WebSocket bus:

1. **`bin/server.py`** — single detached asyncio websockets server per
   port. Started by whichever client wins the `bind()` election. Owns
   the registry of connected agents, mints `msg_id`s, writes
   `messages.log`. Idle-shutdown after N minutes.

2. **`bin/client.py`** — long-lived per-session monitor. Each stdout
   line becomes a Claude Code notification. Manages reconnect with
   exponential backoff, ping/pong, and a ppid-based dedup flock.

3. **`bin/{send,list}.py`** — short-lived control CLIs. Connect with
   `role=control`, do not register as agents, never appear in `list`.
   Discover their owning session via `bin/discover.py` (process-tree
   walk + per-listener state file).

`bin/spawn.py` is the election + spawn boundary; `bin/shared.py` is
paths, validation, sanitizer, atomic bearer-token, identity helpers.

## Non-obvious invariants (read before changing the affected code)

### Race-free server election (`bin/spawn.py` + `bin/server.py --fd`)

The election is `bind()`-atomic: whoever wins spawns the server via
`subprocess.Popen(pass_fds=(fd,), start_new_session=True)`, and the
child adopts the bound fd with `socket.socket(fileno=fd).listen()`.
**PEP 446 is the gotcha**: CPython sets `FD_CLOEXEC` on sockets by
default, so `os.set_inheritable(fd, True)` is required — without it,
`execvp` silently closes the socket. `SO_REUSEADDR=1` is also set to
allow fast rebind after a SIGKILL'd server (otherwise macOS holds the
port for ~30 s).

### Server identity verification (`bin/shared.py::verify_server_identity`)

Before any client or helper sends the bearer token, it verifies the
server process identity by reading the pidfile's `.meta` companion and
checking pid + cmdline + host + port. Refuses on mismatch. This is
defense-in-depth against a coincidental localhost port squatter
receiving the token.

### userConfig is delivered via env vars, NOT `${user_config.*}` substitution

`monitors/monitors.json` deliberately omits `${user_config.*}` because
that substitution breaks `--plugin-dir` local-dev mode (CC doesn't
prompt + substitute in that mode). Instead, CC injects userConfig as
`CLAUDE_PLUGIN_OPTION_*` env vars, and `bin/client.py`'s argparse
defaults read those. **Do not add `--port` or
`--idle-shutdown-minutes` literal CLI args back into `monitors.json`
or the SKILL.md `Monitor` command** — they silently nullify the
user's plugin config. Regression test:
`test_plugin_manifest.py::test_command_does_not_hardcode_userconfig_args`.

### Roles: agent vs control

`role=agent` (client.py, long-lived) appears in `list` and receives
`msg` events. `role=control` (send.py, list.py, ephemeral) does NOT
appear in `list`. Control connections must include `for_session` +
`nonce` matching their owning listener's state file; the server
cross-checks. This blocks impersonation by sibling processes that share
a parent.

### ASCII `name` vs Unicode `label`

`name` is the addressable handle: strict ASCII, regex
`^[a-z0-9][a-z0-9-]{0,39}$`, case-sensitive. `label` is an optional
Unicode display string, NFC-normalized and category-restricted (no
`Cc/Cf/Cs/Cn/Z*`) to block BiDi/ZWJ/NBSP injection. All routing happens
by name; labels are display-only. Don't try to address by label.

### Three-tier size limits

| Limit                          | Value                                       |
| :----------------------------- | :------------------------------------------ |
| WebSocket frame                | 16 MB                                       |
| Direct `text` length           | 10 MB (server-enforced)                     |
| Broadcast `text` length        | 256 KB (server-enforced)                    |
| Stdout notification body       | 400 chars (issue #2: Claude Code clips each monitor notification at ~512 chars total, so the body cap is sized to leave room for our prefix) |

Direct messages whose body exceeds the stdout cap display as a truncated
first-line and a `cont` line pointing to `messages.log` so the receiver
can fetch the full payload via `grep -F <msg_id>`. Truncated content is
preserved in full in `messages.log` regardless.

### Reaction policy lives in `SKILL.md`

The behavioral contract for the receiving agent (when to act, when to
surface, reply prefixes, destructive-op guardrails) is prose in
`SKILL.md`. `tests/test_reaction_policy.py` runs static checks on that
prose so prose edits can't accidentally drop a guardrail.

## Test conventions

- **State isolation**: the `tmp_data_dir` fixture sets
  `INTER_SESSION_DATA_DIR` to a per-test temp path so the suite never
  touches `~/.claude/data/inter-session/`.
- **Free ports**: the `free_port` fixture binds port `0` to find an
  ephemeral port.
- **PPID override**: subprocesses spawned in a single test share the
  pytest parent pid, which would collide on the ppid flock. Set
  `INTER_SESSION_PPID_OVERRIDE` to give each subprocess a distinct
  pseudo-ppid.
- **Slow tests** (`@pytest.mark.slow`): subprocess-spawning, >1 s.

## Don't

- **Don't blanket `pkill -f 'bin/(client|server).py'`** during local
  testing — it will kill real user inter-session monitors running in
  other CC sessions. Target specific pids via the pidfile
  (`~/.claude/data/inter-session/server.<port>.pid`) or
  `TaskList()`-derived monitor task IDs.
- **Don't use `${user_config.*}` substitution in `monitors.json`** —
  see invariant above.
- **Don't weaken the SKILL.md description** ("pushy" multi-trigger
  framing is intentional to combat undertriggering — skill-creator
  best practice).
- **Don't update one README without updating the other.** The repo
  ships both `README.md` (English) and `README.zh.md` (Simplified
  Chinese), cross-linked at the top of each. They're considered a
  pair — when you change one, mirror the change in the other in the
  same commit. If the English content shifts but the Chinese gets
  stale, readers landing from `README.zh.md` get a misleading
  description of how the project works.
- **Don't bump the version in only one of the two plugin manifests.**
  `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
  both carry a `version` field and are consulted by different code
  paths (plugin.json drives installed-plugin update detection;
  marketplace.json drives the marketplace listing). They must stay
  in sync — every version bump touches both files in the same commit.

---
> Source: [yilunzhang/claude-code-inter-session](https://github.com/yilunzhang/claude-code-inter-session) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
