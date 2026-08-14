## canon-music

> Desktop music player + tag management tool for self-hosted music servers. Supports Navidrome. Built Tauri + React. Full spec: `plan.md`. File/data flow map: `ARCHITECTURE.md`.

# Canon

Desktop music player + tag management tool for self-hosted music servers. Supports Navidrome. Built Tauri + React. Full spec: `plan.md`. File/data flow map: `ARCHITECTURE.md`.

**"donow" always means `instructions/donow.md`.** When the user says "donow" (or "do now"), read that file and work its top task.

**Research via explorer agents, never in chat.** When you need to research something (grep reference-projects, map unfamiliar code, locate patterns), delegate it to `caveman:cavecrew-investigator` / Explore agents to keep main-thread context lean. Don't run broad searches inline.

---

## Stack

| Layer | Choice |
|---|---|
| App framework | Tauri — thin Rust layer (audio + keychain only); all business logic TypeScript |
| Frontend | React + TypeScript |
| State | Zustand (playback) + React Query (library data) |
| Local DB | SQLite via `tauri-plugin-sql` |
| Auth storage | OS keychain via `tauri-plugin-keychain` — never disk |
| Audio | `rodio` + `symphonia` in Rust; streaming from server |
| Distribution | GitHub Releases + Tauri auto-updater |

## Commands

```bash
pnpm install                    # install JS deps
pnpm tauri dev                  # run app in dev mode (hot reload)
pnpm tauri build                # production build (.AppImage/.dmg/.msi)
pnpm tsc --noEmit               # typecheck
cd src-tauri && cargo check     # rust typecheck
cd src-tauri && cargo clippy    # rust lint
cd src-tauri && cargo fmt       # rust format
```

---

## Git Workflow

Two branches:

- `development` — all work here. Commit when logical unit done. Auto-commit hook also picks up uncommitted changes on stop.
- `main` — releases only. One commit per release, tagged `vX.Y.Z` by CI. Never commit main direct.

**Always commit after change done**, even if user skip `/next` or `/commit`. Every finished logical unit lands commit before session end/move on.

Release: run `/release`. Skill handle code review, version bump, merge, push.

---

## Architecture Rules

### Keep ARCHITECTURE.md current
Canonical map: every file, purpose, data flow, key invariants. Change that add/move/delete/repurpose file → update `ARCHITECTURE.md` same commit. New Tauri commands, new migrations, new architectural invariants belong there too. Part of "done".

### Rust scope follows reference-project precedent
No hard cap on Rust business logic anymore (former "Rust stays thin, TS-only business logic" rule retired 2026-07-14). Before adding non-trivial logic to `src-tauri/`, check `reference-projects/psysonic` (Tauri v2 + Rust, same category of app) for how it split the same concern — match its precedent (e.g. concurrency/semaphore patterns, connection handling) rather than defaulting to thin Rust or reinventing from scratch. Audio control, OS keychain access, network discovery primitives remain the clearest Rust-native cases regardless. See `.claude/rules/audio-playback.md`.

`discover_upnp_renderers` in `src-tauri/src/upnp.rs` does SSDP UDP multicast discovery — WebKit/JS can't send UDP multicast. Returns raw LOCATION URLs; SOAP control, renderer state management stay TypeScript (`src/lib/dlna.ts`, `src/store/playbackTarget.ts`).

`CoverState` in `src-tauri/src/lib.rs` registers a custom `cover://` URI scheme protocol (`register_asynchronous_uri_scheme_protocol`) for cover art + artist image caching, backed by an in-memory `HashMap<cache_key, (bytes, content_type)>` (capped, clear-on-overflow) plus an on-disk tier under `<app_data_dir>/cover-cache` (capped, oldest-mtime eviction). Registering a scheme handler and serving raw bytes is unavailable to the TypeScript/WebKit layer. Handler pure network primitive — URL construction, credential management, cache-key decisions all TypeScript (`src/lib/navidrome.ts`), which builds `cover://localhost/cover/<id>?size=<n>` and `cover://localhost/artist-image/<encoded>` URLs consumed directly by `<img src>`. No TCP listener involved (unlike the previous `tiny_http` loopback-server design), so the listener/thread-lifecycle risk class in `known-issues.md` does not apply here.

### Enrichment local-only — no file writes
Metadata enrichment (Last.fm tags, artist bio/stats/similar, MusicBrainz identity) writes SQLite only. Canon never modify user's music files. Sidecar file-write subsystem removed; `pending_edits` / `edit_history` tables and `servers.sidecar_*` columns inert legacy schema. File-write design TBD future version.

### Genre tree is DAG
Don't flatten single-parent. Don't merge `canon-tree.json` + `user-tree.json`. See `.claude/rules/genre-tree.md`.

---

## Status

**v0.6.x — active dev.** Schema v20, all workstreams shipped:

- Full library sync (incremental, artists table, tag issues scan)
- Scrobble queue + flush Navidrome (`useScrobbleFlush`)
- Local tag normalization: Last.fm + MusicBrainz genres → canon tree → `album_genres` / `album_unresolved_genres`
- Artist enrichment: bio, stats, similar artists persisted `artist_identity` (schema v20); on-open + background
- Tag issue detection + `TagIssuesView` w/ dismiss + sidebar badge
- Shuffle re-seeds repeat-all wrap
- Drag-to-reorder queue (HTML5 DnD)
- OS media keys (`navigator.mediaSession`, exposes MPRIS Linux)
- Radio Auto-DJ: canon tree ancestor scoring + Last.fm similar artists; 10-track lookahead
- Lyrics: LRClib fetch + SQLite cache + synced auto-scroll NowPlayingOverlay
- Settings: centered layout, unified "Metadata & Tags" section

**Not scope / TBD:** Writing tags back music files (removed, re-design pending), AcoustID fingerprinting, true sample-accurate gapless, streaming HTTP seek, package manager distribution, MusicBrainz submission, light theme.

---

## Detailed Rules (auto-loaded by path)

| File | Loaded when editing |
|---|---|
| `.claude/rules/coding-standards.md` | always |
| `.claude/rules/known-issues.md` | always |
| `.claude/rules/state-management.md` | always |
| `.claude/rules/audio-playback.md` | `src-tauri/**`, `src/store/player.ts` |
| `.claude/rules/tag-editing.md` | `src/db/**`, `src/components/AlbumDetail.tsx` |
| `.claude/rules/genre-tree.md` | `src/assets/**`, `scripts/**` |
| `.claude/rules/sync.md` | `src/lib/**`, `src/hooks/use*.ts` |
| `.claude/rules/server-auth.md` | `src/keychain.ts`, `src/hooks/useServer.ts`, `src/lib/navidrome.ts` |

---
> Source: [jellybrigade/Canon-Music](https://github.com/jellybrigade/Canon-Music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
