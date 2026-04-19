## notionplaylistetl

> Python 3.12 / tkinter desktop app. Fetches Spotify playlists via Spotipy, displays lyrics, exports to Notion (4 databases) or CSV.

# NotionPlaylistETL

Python 3.12 / tkinter desktop app. Fetches Spotify playlists via Spotipy, displays lyrics, exports to Notion (4 databases) or CSV.

## Run & Setup
```bash
source .venv/bin/activate && python main.py
```
Credentials in `.env`: `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `SPOTIFY_REDIRECT_URI`, `NOTION_API_KEY`.

## Module Map

| Module | Purpose |
|---|---|
| `config.py` | Env vars; re-exports DB IDs from `notion_config.py` |
| `notion_config.py` | DB IDs, parent page ID, `DB_NAMES` schema mapping — regenerated per Teamspace by `notion/setup.py` |
| `spotify.py` | `fetch_all_tracks`, `fetch_user_playlists` |
| `lyrics.py` | `fetch_lyrics` with file cache |
| `export.py` | `export_to_csv` |
| `cache.py` | Per-playlist track cache (`tracks/`) |
| `logger.py` | Logging + in-app queue |
| `theme.py` | Dark-mode constants |

### notion/ package
| File | Purpose |
|---|---|
| `__init__.py` | Re-exports: `SKIP`, `export_tracks`, `export_playlist`, `export_playlist_songs`, `fetch_databases`, `snapshot_schema` |
| `_api.py` | `_notion_request`, `_notion_post`, `_notion_get` — raw HTTP with rate-limit retry |
| `_helpers.py` | Shared utilities: `SKIP`, `_page_title`, `_normalize_spotify_url`, `_make_registry_entry`, `_merge_candidates`, `_apostrophe_variants`, `_song_title_variants`, `_chunks` |
| `_artists.py` | Artist CRUD + `_batch_lookup_artists` + `_fetch_artist_details` + `_ensure_artist` (supports `auto_create`) |
| `_songs.py` | Song CRUD + `_batch_lookup_songs` + `_load_all_songs_cache` + `export_tracks` + `_ensure_song` (supports `auto_create`) |
| `_playlists.py` | Playlist CRUD + `_batch_lookup_playlists` + `export_playlist` |
| `_playlist_songs.py` | Playlist song CRUD + lyrics blocks + `_get_playlist_songs_config()` (discovers relation names) + `export_playlist_songs` |
| `_schema.py` | `fetch_databases`, `snapshot_schema` |
| `setup.py` | DB creation from `notion_schema/*.json`, relation wiring, integration test, `update_config_file` |
| `check_setup.py` | `check_notion_setup`, `_reload_config`, `get_db_ids` |

### UI
| Class | File | Purpose |
|---|---|---|
| `App` | `ui/app.py` | Root window; Notebook tabs, Spotify connection, Notion setup orchestration |
| `PlaylistBrowser` | `ui/browser.py` | Playlist listbox; double-click opens `PlaylistTab` |
| `PlaylistTab` | `ui/playlist_tab.py` | Track treeview + lyrics; Export/Refresh buttons |
| `ExportDialog` | `ui/export_dialog.py` | 3-phase export: Playlist → Songs & Artists → Playlist Songs; auto-create prompt |
| `NotionMatchDialog` | `ui/match_dialog.py` | Candidate picker for ambiguous Notion matches |
| `SettingsTab` | `ui/console.py` | Schema verify, Reset Notion Databases button, live log viewer |

## Config & DB ID Loading

DB IDs live in `notion_config.py` → re-exported by `config.py` → imported by `notion/_*.py` modules.

**Stale import problem**: Top-level `from config import NOTION_*_DB_ID` caches the value at import time. After `notion_config.py` is rewritten (setup/reset), `importlib.reload()` updates the module objects but code that already imported values still holds stale references.

**Current state of the fix**:
- `check_setup.py:_reload_config()` reloads `notion_config` → `config` → all `notion._*` modules
- `ui/export_dialog.py` imports `NOTION_PLAYLISTS_DB_ID` inside `_run()` (dynamic)
- `notion/_playlists.py` has `_get_playlists_db_id()` dynamic getter
- **Still using top-level imports**: `_songs.py`, `_artists.py`, `_playlist_songs.py` — these will serve stale IDs after a setup/reset until the app is restarted or modules are reloaded

## Notion Database Setup (`notion/setup.py`)

Triggered on startup via `App._check_notion_setup()` if `check_notion_setup()` finds missing DBs, or via Settings tab "Reset Notion Databases" button.

**Reset flow** (`SettingsTab._reset_notion_databases`):
1. Confirmation dialog
2. Delete all 4 databases + MusicTunnel page via Notion API
3. Reset `notion_config.py` to `"missing"` values (preserves parent page ID)
4. Triggers `App._on_notion_reset_complete()` → `_reload_config()` → `_check_notion_setup()` to offer re-setup

**Setup flow** (`setup_databases_in_page`):
1. Create "MusicTunnel" page under stored parent page
2. Create 4 databases from `notion_schema/*.json` (skips relation/formula/rollup properties)
3. Add relations via raw PATCH API (tracks `dual_property` pairs to avoid duplicates)
4. Verify all properties exist
5. Integration test: create test artist→song→playlist→playlist_song, verify relations, archive test data
6. Write new IDs to `notion_config.py`

**Dual relation handling**: `dual_property` relations auto-create the reverse property on the target DB. The `created_dual_pairs` set (sorted tuple of both DB IDs) prevents adding the same relation from both sides.

**Relation PATCH format**: The Notion API requires the relation type key to appear both as `"type": "dual_property"` AND as a nested object `"dual_property": {}` in the payload. Without the nested object key, the API returns a 400 validation error.

**Integration test**: Discovers actual relation property names by querying the Playlist Songs DB and matching `relation.database_id` to known DB IDs — avoids hardcoding names that Notion may auto-generate differently.

**Dynamic relation discovery** (`_playlist_songs.py`): `_get_playlist_songs_config()` queries the Playlist Songs database schema to find the actual property names for Song and Playlist relations by matching `relation.database_id`. Cached after first call; reset on module reload.

## Export Flow

### Registries (ephemeral per-export)
Temporary dicts tracking created/matched items during a single export. Built via `_make_registry_entry()`. Not persisted.

| Registry key | Format |
|---|---|
| `songs_reg` | Spotify track URL |
| `artists_reg` | Spotify artist ID |
| `playlists_reg` | Spotify playlist ID |
| `pl_songs_reg` | `"{playlist_id}:{track_url}"` |

### Match chain (songs, artists, playlists)
All three `_ensure_*` functions follow the same pattern using shared helpers:
1. **Pre-flight batch lookup** → auto-accept Spotify URL/ID matches (no dialog)
2. **Registry fast-path** → return `"pre_existing"`
3. **Name search** (exact + similar) → `_merge_candidates()` dedup/filter → `NotionMatchDialog`
4. **Backfill** only after user confirms, **create new** if no match

`_backfill_*` functions only run AFTER user confirmation. Never before.

### Auto-create mode
When all tracks in a playlist have Spotify URLs, the export dialog offers to skip name-based matching. Flow:
1. After Phase 1, `ExportDialog` runs a pre-flight URL check to count unmatched songs/artists
2. If unmatched items exist, prompts: "Create new records for all unmatched?" (Yes/No)
3. If confirmed, `auto_create=True` is passed to `export_tracks` and `export_playlist_songs`
4. `_ensure_song` / `_ensure_artist` with `auto_create=True` skip name search + match dialog and create directly when not found by URL/ID

This eliminates per-item match dialogs when Spotify URLs are the definitive identifier.

### Playlist Songs (`export_playlist_songs`)
- Exports missing songs/artists on the fly
- Links playlist, song, and artists via relations
- Lyrics added as two-column Notion block (split at 1900 chars; Notion limit is 2000)
- Idempotent: re-run repairs missing relations on existing records

## Threading

Export runs on a background thread. Match dialogs use this pattern:
1. Background thread calls `match_cb(kind, name, candidates)`
2. `match_cb` schedules dialog on main thread via `self.after(0, ...)`
3. Background thread blocks on `threading.Event.wait()`
4. Main thread shows dialog → sets result → `event.set()`

`SKIP` sentinel = `"__skip__"`.

## Gotchas
- Notion property names use curly apostrophe U+2019 (`'`), not U+0027 — `_apostrophe_variants()` handles search
- Notion rich_text limit: 2000 chars per element
- Notion `files` property: `{"type": "external", "external": {"url": ...}}`
- `_notion_post` requires a body arg; use `_notion_request("GET", ...)` or `_notion_get()` for reads
- 403 on `playlist_items`: private/other-user playlist — handled gracefully
- Notion `dual_property` relations auto-create the reverse; creating from both sides produces duplicates

## Files not in git
`.env`, `.cache`, `playlist_cache.json`, `tracks/`, `lyrics/`, `notion_sync/`, `etl_log_*.txt`, `.venv/`

## Known Issues
- Stale DB ID imports after setup/reset in `_songs.py`, `_artists.py`, `_playlist_songs.py` (top-level imports)
- Duplicate song records possible for items without Spotify URLs
- Match dialog blocks indefinitely (no timeout)
- Export dialog is modal — can't view logs during export

## Future Improvements
1. Fix remaining stale imports (move `from config import NOTION_*` inside functions in `_songs.py`, `_artists.py`, `_playlist_songs.py`)
2. Similarity scoring for match candidates (Levenshtein)
3. Match dialog preview (Spotify vs Notion side-by-side)
4. Auto-timeout on match dialogs
5. Undo/rollback export

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisconnelley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
