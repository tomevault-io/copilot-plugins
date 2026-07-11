## seekify-musicplayer

> Self-hosted music server: Go backend (package main, single binary) serving a vanilla JS/CSS SPA frontend. Scans local music directories, streams audio, downloads via yt-dlp, fetches metadata from MusicBrainz.

AGENTS.md

Project

Self-hosted music server: Go backend (package main, single binary) serving a vanilla JS/CSS SPA frontend. Scans local music directories, streams audio, downloads via yt-dlp, fetches metadata from MusicBrainz.

Build & Run

shgo build -mod=vendor -o server .   # build (vendor mode, no network)
./server                            # run; serves on :8081
go run .                            # build+run in one step

The binary auto-opens a browser. Use -dir flag or MUSIC_DIR env var to set the music directory (defaults to ./music).

Architecture


Single Go package — all .go files at repo root are package main, compiled into one binary.
Entrypoint: server.go (func main()). No separate main.go.
Key files:

server.go — startup, flag parsing, route registration, HTTP server, middleware
handlers.go — HTTP API handlers (largest file)
database.go — SQLite via modernc.org/sqlite (no CGo), WAL mode
scanner.go — music file scanning, tag reading, embedded cover extraction
watcher.go — polling filesystem watcher (file counts, not fsnotify) for live library updates
downloads.go — yt-dlp download job queue, YouTube search scoring, watchdog
musicbrainz.go — MusicBrainz metadata lookup, Cover Art Archive, Deezer artist art, Finder search
waveform.go — audio waveform generation via ffmpeg
models.go — data structs (Track, Album, Playlist, etc.)
settings.go — app settings stored in SQLite key-value table
state.go — shared state helpers
ids.go — ID generation (track/album hashes, UUIDs) — see Hard invariants
review.go — track review worker: auto-flags tracks with missing metadata, suspicious names, duplicates, duration anomalies
autosort.go — moves files into Artist/Album/ structure after metadata scan
watched.go — YouTube playlist watching and auto-downloading
Frontend: index.html, admin.html, ripperv2.html; js/ (vanilla JS, no framework, global module objects loaded via <script> tags); css/ (plain CSS, aggregated by styles.css). Served as static files by the Go server.

js/ui.js — all DOM rendering (largest frontend file)
js/store.js / js/player.js / js/api.js — state, audio playback, API calls
js/review.js — ReviewUI module: now-playing overlay, edit metadata modal, review actions
js/ripperv2.js — standalone ripper/downloader page






Database: data/music.db (SQLite, auto-created on first run, gitignored).


Dependencies


Go module: musicapp (Go 1.26)
Vendor directory committed (vendor/). Always use -mod=vendor or rely on the vendor dir.
Key libraries: github.com/dhowden/tag (audio tag reading), modernc.org/sqlite (pure-Go SQLite).
Runtime external tools (optional but expected in Docker): yt-dlp, ffmpeg, python3 (V2 enrichment).


Environment Variables

VariablePurposeDefaultMUSIC_DIRPrimary music directory./musicMEDIA_MUSIC_DIRSecondary (read-only) music directory, mounted as media: prefix(none)PORTHTTP listen port8081

Hard invariants — never change these

These are data contracts. Breaking them corrupts user data or breaks clients even if the code "looks better" afterward.


ID generation. Track ID = SHA-256(filePath)[:12]. Album ID = SHA-256(lower(artist|album))[:12]. IDs are persisted in favorites, playlists, recents, reviews, download jobs, and cover/waveform cache filenames. Changing the algorithm, input normalization, or truncation orphans existing user data. If a task seems to require changing this, stop and confirm — that needs a data migration plan, not a code edit.
Path prefix scheme. Secondary-library paths are stored with a media: prefix; primary paths have none. The stored format feeds ID generation — preserve it exactly.
dbUpsertTrack preserves tag fields when has_metadata = 1 so the scanner doesn't clobber user-approved metadata. This asymmetry is intentional.
API JSON shapes. The frontend consumes /api/* responses directly. Field names, casing, nesting, and types only change when both sides are updated in the same step and the affected UI is verified.
Schema migration style. CREATE TABLE IF NOT EXISTS + ALTER TABLE ADD COLUMN only (additive). No migration frameworks, no column renames, no semantic changes to existing columns. An existing production DB must open cleanly with new code.
Concurrency. Global maps (tracks, albums, cover/artist-art caches) are sync.RWMutex-guarded and accessed by HTTP handlers plus background goroutines (watcher, download queue, review worker, schedulers). Preserve exact locking behavior; never leak map references outside existing lock patterns; treat any lock-ordering change as high risk and call it out.
Startup behavior. DB-to-memory load order and the optimistic scan-skip (matching file counts → skip rescan) stay as-is.
Graceful degradation. yt-dlp, ffmpeg, and python3 are optional at runtime. Preserve every fallback path (Python enrichment → ffmpeg tagging; cover chain ending in SVG placeholder).


Testing & verification

Characterization tests may exist under *_test.go — never weaken, skip, or delete a test to make a change pass. After every change:


go build -mod=vendor ./... must succeed.
go vet ./... must introduce no new warnings.
go test ./... must pass.
For frontend or handler changes: run the app and manually exercise the affected path (library loads, a track streams, the changed UI works). JS has no type checking or build step — never assume a JS change is safe without loading it.
Review the diff: only the intended change, no drive-by edits.


Working style

Applies to all work — features, bug fixes, and refactors:


Keep diffs scoped to the task. No drive-by cleanup, reformatting, or "while I'm here" changes in unrelated code — propose those separately.
One concern per change. Don't mix a refactor with a bug fix or a feature in the same step; sequence them as separate, individually verified steps.
File layout: splitting or adding files within root-level package main is fine. Moving .go files into subdirectories creates new packages — get approval first. New <script> tags in index.html must respect load order (shared global scope, no module system, no ES imports).
No new frameworks, build steps, or frontend libraries. No new Go dependencies without approval; if approved, run go mod vendor and commit vendor/.
Don't touch Dockerfile, CI, ports, volumes, env vars, or admin auth unless explicitly asked.
Update CHANGELOG.md with every user-facing change (bug fix, feature, UX change). Add one line to the top of the Unreleased block. Commit it in the same commit as the change or immediately after.


Task-specific expectations:


Refactoring: behavior-preserving, mechanical, verbatim-move-first. If unsure whether a refactor changes behavior, stop and ask.
Bug fixes: changing behavior is the point — state old behavior, new behavior, and add a regression test where the logic is testable.
Features: new API routes, additive columns, settings keys, and UI are fair game within the invariants. Follow existing patterns (handler style, mutex discipline, Store/UI module pattern) rather than introducing new ones.


Deployment


GitLab CI (.gitlab-ci.yml): builds Docker image, deploys container on push to main.
Docker image: builds with go build -mod=vendor, runs on Alpine with python3, ffmpeg, yt-dlp pre-installed.
Production mounts: /app/data (DB), /music (primary library), /media-music (secondary library).
Exposed port in Docker: 8081 (mapped to 8298 in CI config).


Gotchas


There is no main.go file. The entry point is server.go.
Vendor mode is required. go.sum and vendor/ are committed. Do not run go mod tidy or go get without also running go mod vendor.
data/ directory and *.db files are gitignored — runtime-only.
The music/ directory contains test audio data (mostly gitignored).
The old app/ and reference apps/ directories are legacy/reference only — do not modify.
The SPA catch-all route serves index.html for any non-/api/ path.
~18 tracked files are flagged git assume-unchanged (index.html, server.go, internal/store/database.go, internal/handlers/handlers.go + settings.go, most js/, css/finder.css, css/review.css, ripperv2.html, etc.). Edits to a flagged file are invisible to `git diff` and silently won't stage. Before staging such an edit run `git update-index --no-assume-unchanged <file>`, then re-flag after commit to preserve the baseline. Check one: `git ls-files -v <file>` (lowercase `h` = flagged, `H` = clean). List all: `git ls-files -v | grep '^[a-z]'`. This has silently dropped core fixes before — always verify `git diff --cached --stat` shows every intended file.

---
> Source: [sadoway7/seekify-musicplayer](https://github.com/sadoway7/seekify-musicplayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
