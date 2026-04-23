## open-packet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                        # install dependencies
uv run pytest                  # run full test suite
uv run pytest tests/test_store # run tests in one directory
uv run pytest tests/test_store/test_store.py::test_list_outbox_excludes_sent_and_deleted  # run single test
uv run open-packet             # run the app (requires config.yaml)
uv run open-packet test.yaml   # run with a specific config file
uv run textual run --dev open_packet/ui/tui/app.py  # run with Textual live-reload + DOM inspector
```

`asyncio_mode = "auto"` is set in `pyproject.toml`, so async tests do not need `@pytest.mark.asyncio`.

## Architecture

The codebase is layered: **Transport → Link → AX.25 → Node → Engine → Store / TUI**. Each layer communicates only with its immediate neighbors via well-defined interfaces.

```
Transport (tcp.py / serial.py)   — raw byte I/O
  └─ Link / KISSLink (kiss.py)   — KISS framing; implements ConnectionBase
       └─ AX25Connection          — AX.25 v2.2 state machine; implements ConnectionBase
            └─ BPQNode            — BBS protocol (L/R/S/K commands); implements NodeBase
                 └─ Engine        — command/event loop running in a daemon thread
                      ├─ Store    — SQLite message persistence
                      └─ TUI      — Textual app, polls Engine event queue every 100ms
```

### Engine (`engine/`)

`Engine` runs in a background thread. The TUI sends it `Command` objects via a `queue.Queue` and reads `Event` objects from a second queue, polled every 100 ms in `OpenPacketApp._poll_events()`. Commands and events are defined in `commands.py` and `events.py`. Adding a new capability means adding a Command, an Event, a `_do_*` handler in `Engine`, and a handler branch in `app._handle_event()`.

### Store (`store/`)

- `models.py` — plain dataclasses: `Operator`, `Node`, `Message`, `Bulletin`
- `database.py` — raw SQLite DDL, schema migrations, and low-level insert/query helpers
- `store.py` — higher-level query methods used by the engine and TUI (`list_messages`, `list_outbox`, `count_folder_stats`, etc.)

Schema migrations use `ALTER TABLE ... ADD COLUMN` in `Database.initialize()` with `except sqlite3.OperationalError: pass`. Always use `self._conn.execute()` for migrations — never `executescript()` (which issues an implicit COMMIT).

**Message state matrix:**
| Type | `queued` | `sent` |
|------|----------|--------|
| Received from BBS | 0 | 0 |
| Composed, awaiting send | 1 | 0 |
| Composed, transmitted | 1 | 1 |

**Bulletin body sentinel:** `Bulletin.body` is `Optional[str]` — `None` means header-only (not yet retrieved). Because the SQLite `body` column has `NOT NULL`, header-only rows are stored as `"\x00"` (NUL byte); `_row_to_bulletin` maps `"\x00"` back to `None`. Do not change this to `""` — empty string is a valid retrieved body. `wants_retrieval: bool` records user intent to fetch the body on the next sync.

The TUI calls `self._store` directly for lightweight operations that don't need connectivity (e.g. `mark_message_read`, `mark_bulletin_wants_retrieval`). Only operations that require a BBS connection go through the engine command queue.

### TUI (`ui/tui/`)

- `app.py` (`OpenPacketApp`) — top-level Textual `App`; owns the engine, store, and active operator; handles all cross-widget coordination
- `screens/main.py` — primary layout; key bindings defined here in `BINDINGS`
- `widgets/folder_tree.py` — uses `node.data` (not `str(node.label)`) for folder routing so count suffixes don't break selection
- `widgets/console_panel.py` — receives `ConsoleEvent` objects from the engine via `log_frame(direction, text)`

Operator identity and node configuration are stored in SQLite and managed via TUI screens, not the config file. The config file (`~/.config/open-packet/config.yaml`) covers connection and storage settings only.

### AX.25 / Node (`ax25/`, `node/`)

`AX25Connection` implements the AX.25 v2.2 data-link state machine (SABM/UA/DISC/I-frames, T1/T3 timers, Go-Back-N windowing). `BPQNode` sits on top and speaks the BPQ32 BBS text protocol. `connect_node()` sends `BBS\r`, waits for a prompt ending in `>`, and handles name prompts by replying with the operator callsign.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/loredous) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
