## herdr-agent-inbox

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [herdr](https://herdr.dev) plugin (stdlib Python only, no dependencies) that
turns herdr's agent sidebar into an inbox: transcript-derived session titles,
settle/mark-unread triage ordering, per-agent runtimes, workspace rollups, and
an accumulated, resumable chat history. Everything drives herdr through its
public socket API and plugin manifest — never patch or assume herdr internals;
verify against the installed CLI (`herdr --help`, `herdr api schema --json`).

## Commands

```sh
python3 -m py_compile daemon.py actions.py inbox_tui.py   # syntax gate (no test suite yet)
python3 inbox_tui.py --demo          # run the popup standalone with canned data (no herdr needed)
sh scripts/restart-daemon.sh         # kill + restart the daemon (verifies pid, waits for flock)
herdr plugin link "$PWD"             # (re)register with a live herdr — rereads herdr-plugin.toml
herdr plugin action invoke settle --plugin herdr-agent-inbox   # exercise an action end-to-end
tail -f "$(dirname "${HERDR_SOCKET_PATH:-$HOME/.config/herdr/herdr.sock}")/agent-inbox-state/daemon.log"
```

Manifest changes (`herdr-plugin.toml`) only take effect after re-linking.
Daemon code changes only take effect after `scripts/restart-daemon.sh`.

## Architecture

Three processes share one contract:

- **`daemon.py`** — the only long-lived process and the only writer of state.
  Threads: `events_loop` (persistent `events.subscribe` connection; herdr
  closes every other request's connection after one response — see
  `herdr_request`), `control_loop` (unix socket serving one-line JSON commands
  from the other two programs), `summarize_loop` (serial queue shelling out to
  the user's `summarize_cmd`), and the main loop (debounced `refresh()`).
  `refresh()` diffs `agent.list` against `self.terminals` (keyed by
  `terminal_id`, which survives herdr restarts), derives titles, computes the
  inbox `rank` token, and reports pane/workspace metadata — computation under
  `self.lock`, socket I/O outside it (keep it that way; control clients have
  3s timeouts). The sidebar ordering itself is herdr's `agent.view.set`
  sorting by our `rank` token; it and all reported metadata die with the
  herdr server, so every events-loop reconnect clears the report caches and
  re-asserts the view.
- **`actions.py`** — short-lived dispatcher behind the manifest `[[actions]]`;
  forwards ops to the control socket (starting the daemon if needed). The
  sidebar resize is special: herdr's live width is session state that config
  `sidebar_width` cannot move, but min/max clamps re-apply on every
  `reload-config` — so resize pins width=min=max, reloads, then relaxes the
  bounds so herdr's native mouse drag keeps working. Serialized via flock;
  never leave bounds pinned.
- **`inbox_tui.py`** — the popup TUI (curses). Reads herdr directly for live
  rows, `history.jsonl` for archived chats, and sends mutations through the
  control socket. All rendering must go through `seg()`/`_wtrunc()` (display-
  width aware — emoji are two columns; naive `len()`/`ljust` crashes curses at
  the last cell). Colors come from `_THEME_PALETTES`, extracted from herdr's
  own themes; state colors follow herdr's language (blocked=red,
  working=yellow, done-unseen=teal, idle=green). `--demo` swaps the data layer
  for fixtures — keep it working; it's how screenshots and manual tests run.

Shared conventions all three must agree on:

- **Paths derive from the session socket** (`HERDR_SOCKET_PATH`, defaulting to
  `~/.config/herdr/herdr.sock`): state dir is `<socket dir>/agent-inbox-state/`.
  Deliberately NOT `$HERDR_PLUGIN_STATE_DIR` — the daemon may be started from
  a shell without plugin env, and all entry points must resolve identical
  paths.
- **Control protocol** ops (`settle`, `unread`, `clear`, `retitle`,
  `set-title`, `settle-workspace`, `ping`) live in `handle_command`; the
  control thread must never die (it answers malformed input, never raises).
- **`herdr_request` raises only `RuntimeError`/`OSError`** — callers catch
  exactly those. Don't let `ValueError` leak from JSON parsing; a herdr
  restart mid-request must never kill a thread.
- **Rank tiers** (`rank_for`): blocked=0, done/unread=1, working=2, idle=3,
  unknown=4, settled=5. Settling clears on transition INTO working/blocked;
  settled outranks done so an explicitly settled agent stays sunk after
  finishing.
- **Archiving**: `_archive_chat` appends to both the per-terminal history (⚫
  rows in the tree, capped 10) and `history.jsonl` (global browser + resume,
  compacted atomically). Archive triggers: session-ref change, agent-kind
  change, pane gone >120s. Resume commands are built only from
  `sess_kind`/`sess_value` via `resume_cmd()` with `shlex.quote`.

## Constraints

- Stdlib only; `tomllib` (3.11+) is optional — config parsing must degrade to
  defaults, never crash, on older Pythons.
- Titles and tokens are display-only metadata reported under the single
  source id `herdr-agent-inbox`; herdr normalizes/caps them (80 chars, 16
  tokens per report). `_clean_title` strips command wrappers, system
  reminders, and URLs — user prompts are untrusted input.
- Any file the daemon rewrites in place (state.json, history compaction,
  config.toml in actions.py) must write tmp + `os.replace`, and config writes
  must go through `os.path.realpath` so dotfiles symlinks survive.

---
> Source: [douglascorrea/herdr-agent-inbox](https://github.com/douglascorrea/herdr-agent-inbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
