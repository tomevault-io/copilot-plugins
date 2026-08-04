## theoracle

> Manages the pre-draft lobby and holds live `DraftArbiter` instances after sessions start.

# TheOracle — Project Reference

## Overview
MTG draft tracking website backend. Two core capabilities: (1) resolves OCR-scanned card identifiers to full card data via Scryfall, and (2) manages booster draft sessions end-to-end. Exposed via a FastAPI server with a Jinja2 stats website. N player apps send OCR-scanned picks to the server; the server records all state and serves live stats via browser.

## Stack
- **Language:** Python 3.12+
- **Package layout:** src layout — `src/theoracle/`
- **Build backend:** hatchling (`pyproject.toml`)
- **HTTP client:** requests
- **External API:** Scryfall (`https://api.scryfall.com`)
- **Web framework:** FastAPI + uvicorn
- **Templates:** Jinja2 (server-rendered HTML)
- **Test framework:** pytest 9+
- **Environment:** conda env named `oracle`

## Project Structure
```
src/theoracle/
    __init__.py
    card_parser.py        # OCR normalization + Scryfall lookup pipeline
    draft_arbiter.py      # MTG booster draft session manager
    db.py                 # SQLite connection factory + migration runner
    session_manager.py    # Lobby state + SessionManager registry
    main.py               # FastAPI app — API routes + website routes
    templates/
        base.html         # shared layout (Bootstrap 5)
        session_stats.html  # live draft view, auto-refreshes every 5s
        player_history.html # per-player historical picks + pack replay
        cards.html          # top 50 cards by avg pick order
migrations/
    001_initial.sql       # sessions, players, picks, results
    002_global_players.sql  # persistent player identity across sessions
tests/
    test_card_parser.py   # 47 unit tests, all mocked (no network)
    test_draft_arbiter.py # 54 unit tests, SQLite via tmp_path
    test_api.py           # 29 FastAPI tests, mocked card_parser
pyproject.toml            # dependencies, build config, pytest config
```

## Install & Run
```bash
# One-time setup (run from project root)
conda activate oracle
pip install -e ".[dev]"

# Run the server (default DB: drafts.db in cwd)
uvicorn theoracle.main:app --reload

# Override DB path
ORACLE_DB_PATH=/path/to/drafts.db uvicorn theoracle.main:app --reload

# Swagger UI (interactive API docs)
open http://localhost:8000/docs

# Run tests
pytest
```

---

## Module: `theoracle.card_parser`

### Public API
```python
from theoracle.card_parser import parse_card_identifier, CardData

card: CardData | None = parse_card_identifier("M21/123")
```

### `CardData` fields
| Field | Type | Source |
|---|---|---|
| `name` | str | Scryfall `name` |
| `mana_cost` | str | Scryfall `mana_cost` (face 0 for DFCs) |
| `type_line` | str | Scryfall `type_line` |
| `oracle_text` | str | Scryfall `oracle_text` (face 0 for DFCs) |
| `image_url` | str | `image_uris['normal']` (face 0 for DFCs) |

### Pipeline (in order)
1. **`_extract_set_and_number(raw)`** — permissive regex, accepts OCR-mangled input. Set code: 3–5 alphanumeric chars. Separator: `/`, `-`, or whitespace. Collector number: must start with a digit-like char.
2. **`_correct_collector_number(raw)`** — substitutes OCR digit-lookalikes (`O→0`, `l→1`, `I→1`, `Z→2`, `z→2`, `S→5`, `B→8`). Preserves valid Scryfall variant suffixes (`a`–`y` excluding OCR-mapped chars). Lowercase `s` is never corrected (showcase suffix).
3. **`_fuzzy_match_set(raw_set)`** — validates set code against Scryfall `/sets` list using `difflib.get_close_matches` (cutoff 0.6). Falls back to uppercased original if no match.
4. **`fetch_card(set_code, collector_number)`** — primary: `GET /cards/{set}/{number}`; fallback on 404: `GET /cards/search?q=set:X cn:Y`. Returns `None` if both miss.

### Scryfall set code cache
- Path: `src/theoracle/.scryfall_sets_cache.json`
- TTL: 30 days
- On stale/missing: fetches `GET /sets`, rewrites cache
- On network failure: degrades to stale cache data, or `[]` if no cache exists

### Error behaviour
- `RuntimeError` raised on: HTTP 429 (rate limited), HTTP 5xx (server error), network failure
- Non-404/non-200 primary responses abort without fallback (avoids misleading results)

---

## Module: `theoracle.draft_arbiter`

One `DraftArbiter` instance per live draft session. Holds all live state in RAM; persists to a SQLite database via `save()`. Thread-safe — a single `threading.RLock` serialises all mutations. Pack contents are never known up front; they are reconstructed from the ordered pick history after a pack is fully drafted.

### Architecture decisions
- **One arbiter per session** — correct granularity for a web backend (route by `session_id`, each arbiter owns its lock).
- **Session registry lives in `SessionManager`** — `session_manager.py` owns the lobby lifecycle and maps `session_id → DraftArbiter` once a session starts.
- **SQLite persistence via raw SQL migrations** — schema lives in `migrations/`; `db.py` applies migrations automatically on connection open. Migration history tracked in `schema_migrations` table.
- **Explicit round advancement** — `record_pick` never auto-advances rounds. The host calls `POST /sessions/{id}/advance` after `is_round_complete()`, giving the web layer a natural gate for "round over" UI.

### Public API
```python
from theoracle.draft_arbiter import DraftArbiter, PickEvent, PlayerStats, CardStats

arbiter = DraftArbiter(
    session_id="draft-2026-06-20",
    num_players=8,
    pack_size=15,                        # default
    rounds=["left", "right", "left"],    # default
    player_names=["Alice", "Bob", ...],  # default: ["seat_0", ..., "seat_N-1"]
    db_path="drafts.db",                 # optional; save() raises if None
)

event: PickEvent = arbiter.record_pick(seat=0, card_name="Lightning Bolt")

if arbiter.is_round_complete():
    arbiter.advance_round()

arbiter.record_result(winner_seat=2)
arbiter.save()

arbiter2 = DraftArbiter.load("drafts.db", "draft-2026-06-20")
```

### Data classes
| Class | Key fields |
|---|---|
| `PickEvent` | `pack_id`, `seat`, `pick_index`, `card_name` — **frozen** (immutable record) |
| `LogicalPack` | `pack_id`, `round_number`, `origin_seat`, `pack_size`, `pass_direction`, `picks` |
| `PlayerStats` | `player_name`, `picked_cards: list[str]` (primary), `wins`, `losses`, `win_rate`, `card_count(name)` |
| `CardStats` | `card_name`, `times_selected` — computed from DB on demand |

### Pack identity
`pack_id = f"R{round_number}S{origin_seat}"` — e.g. `"R0S2"` is the pack that originated at seat 2 in round 0.

### Pass directions
- `"left"` → next seat = `(current - 1) % num_players`
- `"right"` → next seat = `(current + 1) % num_players`

### Reconstruction methods (require pack to be complete)
```python
arbiter.get_full_pack("R0S2")              # all N picks in order
arbiter.get_pack_before_pick("R0S2", k)   # full_pack[k:]
arbiter.get_pack_after_pick("R0S2", k)    # full_pack[k+1:]
arbiter.replay_draft()                     # list[PickEvent] in recording order
```

### Stats
- `get_player_stats(seat)` / `get_all_player_stats()` — from RAM; `picked_cards` is the ordered pick history (primary).
- `get_card_stats(name)` / `get_all_card_stats()` — queries DB for other sessions, combines with current session RAM (no double-counting).

### SQL schema (SQLite)
Six tables: `global_players`, `sessions`, `session_rounds`, `players`, `picks`, `results`. Multiple sessions share one DB file. Key design choices:
- `global_players` — maps `player_id (UUID) → player_name`; ensures same player gets same ID across sessions
- `picks.pick_index` — 0-based position within a pack (enables avg-pick-order queries directly)
- `picks.player_pick_num` — 0-based sequential pick number for that player within the session (stored at save time; avoids window functions for "first X picks" queries)
- `picks(round_number, origin_seat)` decompose `pack_id` for queryable pack reconstruction
- Indexes: `(card_name, pick_index)`, `(session_id, seat, player_pick_num)`, `(session_id, round_number, origin_seat)`, `(session_id, winner_seat)`

### Error behaviour
- `ValueError` on all structural violations (invalid seat, round not complete, pack not complete, etc.)
- `get_card_stats` returns `None` for unknown cards (not an error)
- No card name legality validation (structural correctness only)

---

## Module: `theoracle.session_manager`

Manages the pre-draft lobby and holds live `DraftArbiter` instances after sessions start.

### Data classes
- **`LobbyPlayer`** — `player_id`, `player_name`, `is_host`
- **`LobbySession`** — `session_id`, `host_player_id`, `num_players`, `pack_size`, `rounds`, `db_path`, `players`, `status` (`"waiting"` | `"active"` | `"complete"`), `seat_map` (`player_id → seat`), `arbiter`, `lock`

### `SessionManager` public API
```python
manager = SessionManager()

session_id, player_id = manager.create_session(player_name, num_players, pack_size, rounds, db_path)
player_id = manager.join_session(session_id, player_name, db_path)
seat_map  = manager.start_session(session_id, host_player_id)   # randomly assigns seats
session   = manager.get_session(session_id)
ok        = manager.verify_player_seat(session_id, player_id, seat)
```

### Player identity
`player_name` is the stable global key. First time a name appears, a UUID4 `player_id` is generated and stored in `global_players`. Subsequent sessions with the same name return the same `player_id`, enabling cross-session history on the website.

---

## Module: `theoracle.main` — FastAPI App

Single `SessionManager` instance + `DB_PATH` (env var `ORACLE_DB_PATH`, default `drafts.db`) shared across all requests.

### API Routes (JSON — for player apps)

| Method | Path | Who | Description |
|--------|------|-----|-------------|
| `POST` | `/sessions` | Host | Create session. Body: `player_name, num_players, pack_size, rounds`. Returns `session_id, player_id`. |
| `POST` | `/sessions/{id}/join` | Player | Join lobby. Body: `player_name`. Returns `player_id`. |
| `GET`  | `/sessions/{id}` | All | Lobby/session state. Poll this during lobby to detect when session starts and seats are assigned. |
| `POST` | `/sessions/{id}/start` | Host | Lock roster, randomly assign seats, begin draft. Body: `player_id`. Returns `seat_assignments`. |
| `POST` | `/sessions/{id}/picks` | Player | Submit pick. Body: `player_id, seat, card_id`. Server resolves `card_id` via `parse_card_identifier`. Returns `card_name, pack_id, pick_index, seat`. |
| `POST` | `/sessions/{id}/advance` | Host | Advance to next round (only when round is complete). Body: `player_id`. |
| `POST` | `/sessions/{id}/results` | Host | Record match result. Body: `player_id, winner_seat`. |

### Error codes
| Code | Meaning |
|------|---------|
| 404 | Session or player not found |
| 403 | `player_id` doesn't match `seat`, or action requires host |
| 409 | Action invalid for current session state (wrong status, session full, name taken) |
| 422 | Card not identified — retry; or `ValueError` from `DraftArbiter` |
| 503 | Scryfall unavailable |

### Website Routes (HTML)

| Path | Description |
|------|-------------|
| `/sessions/{id}/stats` | Live session stats. Auto-refreshes every 5s while active. |
| `/players/{player_id}` | Player's full draft history — pack seen at each pick + card chosen. |
| `/cards` | Top 50 cards by average pick order (highest = taken latest). |

### Testing pattern
```python
# Inject fresh manager + tmp DB per test
app.dependency_overrides[get_manager] = lambda: SessionManager()
app.dependency_overrides[get_db_path] = lambda: str(tmp_path / "test.db")
```

---

## Code Style Rules
- No comments unless the WHY is non-obvious
- No type annotations on local variables — only on function signatures
- All external I/O (network, file, time) must be mockable — no bare `requests.get` calls outside `_safe_get`
- Tests must never hit the network — patch `theoracle.card_parser.requests.get` and `theoracle.card_parser.time.sleep`
- `autouse` fixture `no_sleep` suppresses `time.sleep` globally in `test_card_parser.py`
- Draft arbiter tests use `tmp_path` fixture for SQLite I/O; no mocking needed (pure in-memory except persistence tests)
- API tests use `TestClient` with dependency overrides for `get_manager` and `get_db_path`; patch `theoracle.main.parse_card_identifier` for pick tests

---

## Next Session Action Items

1. **Understand the APIs and DraftArbiter structure** — Read through `draft_arbiter.py`, `session_manager.py`, and `main.py` before making changes. Key things to internalize: how `LobbySession` transitions from `waiting → active → complete`, how `DraftArbiter` assigns packs to seats and passes them, and how `save()` + `load()` work. The Swagger UI at `/docs` (run the server first) is the fastest way to see the full API surface interactively.

2. **Share the API spec with the app developer** — The player app needs to call these endpoints in order:
   - `POST /sessions` (host only, at session creation)
   - `POST /sessions/{id}/join` (each player, returns their persistent `player_id`)
   - `GET /sessions/{id}` — poll until `status == "active"` and `seat` is assigned
   - `POST /sessions/{id}/start` (host only, once all players have joined)
   - `POST /sessions/{id}/picks` — body: `{ player_id, seat, card_id }` — on 422 with "retry", re-scan
   - `POST /sessions/{id}/advance` (host only, after round is complete)
   - `POST /sessions/{id}/results` (host only, to record match outcomes)
   The Swagger UI at `http://localhost:8000/docs` shows full request/response schemas. Direct the app dev there.

3. **End-to-end smoke test with dummy apps** — Write a small script (or set of `curl`/`httpx` calls) that simulates N players sending picks concurrently, runs through a full draft, and checks the stats website renders correctly. Good test parameters: `num_players=4, pack_size=5, rounds=["left","right","left"]`. Verify the `/sessions/{id}/stats` page updates live, and `/players/{player_id}` shows the correct pack history after the session completes.

---

## Planned Next Steps

### Priority 2 — End-to-end smoke test (see Action Item 3 above)

### Priority 3 — Third Scryfall fallback
`GET /cards/named?fuzzy={name}` when OCR also emits a card name (marked TODO in `fetch_card`).

---
> Source: [J4s0nZhang/TheOracle](https://github.com/J4s0nZhang/TheOracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
