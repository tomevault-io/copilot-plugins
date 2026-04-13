## srrb

> **srrb** — Static RSS Reader Backend. Go CLI that fetches RSS/Atom/RDF feeds into gzip-compressed pack series (idx/, data/, ts/). Backends: local filesystem, S3, SFTP.

# CLAUDE.md

## Project

**srrb** — Static RSS Reader Backend. Go CLI that fetches RSS/Atom/RDF feeds into gzip-compressed pack series (idx/, data/, ts/). Backends: local filesystem, S3, SFTP.

## Commands

```bash
go build -o srrb .          # Build
go test ./...               # All tests
go test -run TestName .     # Single test
go test -v ./backend/       # Package tests
```

Release: `CGO_ENABLED=0 go build -ldflags "-s -w"`. No Makefile/linter/Dockerfile.

## Architecture

- **`main.go`** — CLI via `alecthomas/kong` + YAML config (`$SRR_CONFIG` or `$XDG_CONFIG_HOME/srr/srr.yaml`). `Globals` struct for flags. Inline `version` subcommand.
- **`cmd_fetch.go`** — `signal.NotifyContext` for graceful shutdown, channel-based worker pool (`globals.Workers` goroutines). Articles sorted by published time (ascending) before storage. Order: `PutArticles` → `UpdateTS` → `Commit`.
- **`feed.go`** — Streaming XML parser, auto-detects RSS/Atom/RDF. GUIDs: FNV-32a → `uint32` (fallback: GUID → ID → Link → empty hash).
- **`cmd_subs.go`** — `AddCmd` (add/update subscription via `--upd`, `-t/--title`, `-u/--url`, `-g/--tag`, `-p/--parsers`), `RmCmd`, `LsCmd` (filter by `-g/--tag`, yaml/json output).
- **`cmd_import.go`** — OPML import with hierarchical ID selection (`-a` all, `-i` specific). OPML group hierarchy auto-resolves to hierarchical tags; `-g/--tag` overrides. `-n/--dry-run` lists resulting subscriptions without importing.
- **`cmd_preview.go`** — `preview` subcommand: fetches a feed URL, runs module pipeline (`-p`), serves rendered articles via local HTTP server (`-a/--addr`).
- **`subscription.go`** — `Subscription` struct (ETag, Last-Modified, StopGUID, Tag, Pipeline). `Fetch` method handles per-subscription HTTP fetch, `StopGUID`/`ErrStopFeed` incremental logic, and module pipeline execution. Also contains `processItem` and text sanitization helpers.

### Backend (`backend/`)

Low-level storage interface: `Get`/`Put`/`AtomicPut`/`Rm`/`Close`. Registry selects by URL scheme; local = empty scheme `""`. Config registry: backends call `RegisterConfig` in `init()` with a config struct pointer; `LoadConfigs` reads YAML sections into them.

| Method | Behavior |
|---|---|
| `Get(ignoreMissing)` | Controls error-on-missing |
| `Put(ignoreExisting)` | Controls overwrite vs exclusive create |
| `AtomicPut` | temp-then-rename (local/SFTP); overwrite (S3) |

- **`local.go`** — Auto-creates subdirs via `os.MkdirAll`.
- **`s3.go`** — `IfNoneMatch` precondition headers + CRC32 checksums. `S3Config`: region, endpoint, profile, static credentials.
- **`sftp.go`** — Auth chain: URL password → config password → config/default private key → `~/.ssh/` keys → SSH agent → error. Host key verification via `~/.ssh/known_hosts` by default (`SFTPConfig.Insecure` to skip).

### Pack Storage (`db.go`)

`PutArticles`, `UpdateTS`, `savePack` manage three gzip-compressed series:

| Series | Format | Split rule |
|---|---|---|
| `idx/` | TSV metadata (7 columns, see `savePack`) | Every 1000 articles (`idxPackSize`) |
| `data/` | Null-byte-separated content | At `PackSize` (tracked by `next_pid`/`pack_off`) |
| `ts/` | TSV delta snapshots, finalized weekly by epoch-week | By week (epoch / 604800) |

- File-based locking (`.locked` nil-payload marker); lock removal uses `context.WithoutCancel` to survive cancellation.
- `Commit` serializes `DBCore` via `AtomicPut`. `db.json` uses short JSON keys (`data_tog`, `pipe`, `ferr`, etc.) — read `DBCore` struct tags for full schema.
- `data_tog`/`ts_tog` toggle alternating pack filenames for atomic updates.
- `first_fetched` (`FirstFetchedAt`): unix timestamp of the first fetch that produced articles. Clients derive the earliest ts/ week as `first_fetched / 604800` to avoid requesting weeks before any data exists.
- ts/ format: first line per pack is absolute `0 \t TotalArticles [\t subID \t subTotalArticles \t subLastAddedAt]*` (first field always 0, week encoded in filename); subsequent lines are `deltaTS \t TotalArticles [\t subID \t subTotalArticles]*` where `deltaTS` is `FetchedAt % 604800` (absolute week offset) and all counts are absolute (no subLastAddedAt in delta lines). Week-start snapshots include all subs and finalize previous pack; mid-week only dirty subs; multi-week gaps produce one finalized pack per missing week.

### Module System (`mod/`)

Pipeline per-subscription during fetch. Factory pattern: `New()` returns fresh stateful processor.

- `#sanitize` — bluemonday, content-only.
- `#minify` — tdewolff/minify, content-only.
- External modules: `/bin/sh -c`, stdin/stdout JSON (`RawItem`), stderr passthrough. GUID is immutable (change = error).

## Constraints (preserve when editing)

- File-based DB lock (`.locked`) with `--force` override
- Env vars prefixed `SRR_`
- `ErrStopFeed` sentinel halts feed on `StopGUID` match

## AI Agents

- **`claudemd-sync-checker`** — after significant changes, audit CLAUDE.md for drift.
- **`backend-conformance-checker`** — when adding/modifying backends, audit against `Backend` interface.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gllera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gllera)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
