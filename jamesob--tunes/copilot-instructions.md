## tunes

> A self-hosted music server with a power-user web interface. Features multi-pane

# Tunes

A self-hosted music server with a power-user web interface. Features multi-pane
browsing, fuzzy search, audio fingerprinting for duplicate detection, and gapless
playback.

Includes rich metadata like waveform and album art. Provides a nice interface for
updating track metadata, organizing music, and (eventually) pushing to Rekordbox USBs.

## Quick Reference

| Component | Tech | Entry Point |
|-----------|------|-------------|
| Frontend | React + TypeScript + Vite | `frontend/src/main.tsx` |
| Backend | Go + Chi + SQLite | `backend/cmd/tunes/main.go` |
| Database | SQLite with FTS5 | `./data/tunes.db` |
| Config | YAML | `./config.yaml` |

## Development

```bash
# Start dev environment (hot reload enabled)
docker-compose up

# Frontend: http://localhost:5173
# Backend:  http://localhost:8080
```

## Project Structure

```
├── frontend/src/
│   ├── api/client.ts          # REST API client
│   ├── components/panes/      # UI panes (FileBrowser, Playlist, DevLog, etc.)
│   ├── stores/                # Zustand state (player.ts, panes.ts, queue.ts)
│   └── types/api.ts           # TypeScript interfaces
├── backend/internal/
│   ├── api/handlers.go        # HTTP handlers
│   ├── api/router.go          # Route definitions
│   ├── db/db.go               # Schema and migrations
│   ├── db/queries.go          # Database queries
│   ├── indexer/scanner.go     # Music file scanner
│   ├── fingerprint/           # Duplicate detection (chromaprint)
│   └── waveform/              # Waveform generation (ffmpeg)
└── docker/                    # Dockerfiles
```

## Key Files by Task

**Adding a new API endpoint:**
1. `backend/internal/api/router.go` - Add route
2. `backend/internal/api/handlers.go` - Add handler
3. `frontend/src/api/client.ts` - Add client method
4. `frontend/src/types/api.ts` - Add types if needed

**Adding a new pane type:**
1. `frontend/src/components/panes/` - Create pane component
2. `frontend/src/stores/panes.ts` - Add pane type to store

**Modifying database schema:**
1. `backend/internal/db/db.go` - Update schema constant and Migrate()

**Adding new track metadata:**
1. `backend/internal/db/db.go` - Add column
2. `backend/internal/models/models.go` - Add field to Track struct
3. `backend/internal/indexer/scanner.go` - Extract metadata during scan
4. `frontend/src/types/api.ts` - Add to Track interface

## API Endpoints

### Tracks
- `GET /api/tracks` - List tracks (pagination: `?limit=&offset=`)
- `GET /api/tracks/{id}` - Get track
- `GET /api/tracks/{id}/stream` - Stream audio
- `GET /api/tracks/{id}/waveform` - Get waveform data
- `POST /api/tracks/move` - Move file

### Browse & Search
- `GET /api/browse?path=` - Directory listing
- `GET /api/search?q=&limit=` - Full-text search

### Library
- `GET /api/albums`, `GET /api/albums/{id}`, `GET /api/albums/{id}/art`
- `GET /api/artists`, `GET /api/artists/{id}`
- `GET /api/genres`, `GET /api/genres/{name}/tracks`

### Playlists
- `GET/POST /api/playlists`
- `GET/PUT/DELETE /api/playlists/{id}`
- `POST/PUT /api/playlists/{id}/tracks`

### System
- `GET /api/system/status` - Library stats, tool availability
- `POST /api/system/scan` - Trigger rescan
- `POST /api/system/backfill-waveforms` - Generate missing waveforms
- `POST /api/system/backfill-fingerprints` - Generate fingerprints, detect duplicates
- `GET /api/duplicates` - List duplicate tracks
- `GET /api/events` - SSE stream for real-time updates

## Database Schema (SQLite)

Core tables: `tracks`, `albums`, `artists`, `playlists`, `playlist_tracks`, `play_history`

Key track columns:
- `file_path`, `file_name`, `file_size`, `file_modified`
- `title`, `artist`, `album`, `album_artist`, `genre`, `year`
- `track_num`, `disc_num`, `duration_ms`, `format`, `bitrate`
- `waveform` (JSON array), `file_hash` (SHA256), `audio_fingerprint` (chromaprint)
- `duplicate_of` (FK to canonical track)

Full-text search: `tracks_fts` virtual table (FTS5)

## External Dependencies

- **ffmpeg/ffprobe**: Waveform generation, duration extraction
- **fpcalc** (chromaprint): Audio fingerprinting for duplicate detection

Both installed in Docker containers via Alpine packages.

## Code Conventions

- **Frontend**: React functional components, Zustand for state, TanStack Query for data fetching
- **Backend**: Chi router, handler functions in handlers.go, database queries in queries.go
- **Styling**: Tailwind CSS, dark mode via system preference, minimal borders, monospace accents
- **Comments**: Sparse - prefer self-documenting code, function-level docs only when needed

## Style preferences

- Keep code simple as can be, avoid lots of documentation unless helpful to LLMs.
  - When doing doc, prefer module and function-level doc.
- Avoid frivolous inclusion of third party dependencies when possible (e.g. if a
  library can easily be substituted with a use of the standard libraries), but take
  dependencies when they make sense -- like for sophisticated audio analysis.

- The UI should be simple, modern, efficient, and aimed at power users. Think an
  interface like Linear or Sentry.

## Implementation Status

### Completed
- Multi-pane interface with file browser, albums, artists, genres, playlists
- Fuzzy search with Ctrl+K command palette
- Audio playback with queue management
- Waveform visualization
- Drag-and-drop for panes and playlist management
- Audio fingerprinting and duplicate detection
- Real-time event log (DevLog pane)
- Dark mode support

### Planned
- Gapless playback (optimistic next-track streaming)
- MusicBrainz metadata lookup
- Multi-user support (schema exists, UI assumes single user)
- Album art auto-fetch from external sources

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesob)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jamesob)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
