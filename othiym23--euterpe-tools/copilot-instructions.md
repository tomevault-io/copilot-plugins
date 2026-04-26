## euterpe-tools

> Incremental CLI filesystem scanner and audio metadata manager for a Synology

# euterpe-tools

Incremental CLI filesystem scanner and audio metadata manager for a Synology
NAS. See [docs/DESIGN_NOTES.md](docs/DESIGN_NOTES.md) for architecture details.

## Repository layout

```
crates/          Rust libraries (etp-lib, etp-cue)
cmd/             All plumbing commands, any language
  etp-csv/       Rust — CSV output
  etp-tree/      Rust — tree output
  etp-find/      Rust — regex file search
  etp-meta/      Rust — metadata scan/read/cue
  etp-scan/      Rust — standalone directory scanner
  etp-cas/       Rust — CAS blob operations
  etp-query/     Rust — database queries
  etp/           Python — porcelain dispatcher + anime + catalog
pylib/           Python shared library (etp_lib)
conf/            KDL configuration files
docs/            Plans, ADRs, design notes
```

## Build & run

Most development happens inside a devcontainer (see `.devcontainer/`). The
container has the full Rust and Python toolchain for building, checking, and
testing. However, `just build-nas` and `just deploy` require the macOS
cross-compilation toolchain and write access to NAS shares, so they must be run
on the host — do not attempt them inside the devcontainer.

```bash
just build-smoketest  # verify all crates compile
just build            # native release (aarch64-apple-darwin)
just build-nas        # NAS release (x86_64-unknown-linux-musl, static) — host only
just deploy           # check + test + build + copy to NAS — host only

# Rust plumbing (in cmd/)
etp-scan <directory> [--db <path>] [-e <name>...] [-v]
etp-csv <directory> [--scan] [--output <file.csv>] [--db <file.db>] [-a] [--exclude <name>...] [--[no-]include-system-files] [-v]
etp-tree <directory> [--scan] [--db <file.db>] [-a] [--exclude <name>...] [--[no-]include-system-files] [--du [--du-subs]] [-v]
etp-find <pattern> [-R <directory>] [--tree=<file>] [--csv=<file>] [--size] [-a] [-i] [--db <path>] [--[no-]include-system-files] [-v]
etp-init [--force]
etp-meta scan [-R <directory>] [--db <path>] [-e <name>...] [--force] [-v]
etp-meta read <file> [--images]
etp-meta cue <file> [--audio-file PATH...] [--format summary|cuetools|eac]
etp-cas store <file>
etp-cas get <hash> [-o PATH]
etp-cas gc --db <path> [-v]
etp-cas list
etp-query --db <path|nickname> files|tags|find|stats|size|sql [args...]
etp-query stats [--format text|csv|json|kdl] [--[no-]include-system-files]

# Python porcelain (in cmd/etp/)
etp tree <directory> [args...]
etp find <pattern> [-R <directory>] [args...]
etp catalog [--dry-run] [config.kdl]
etp anime triage|series|episode [args...]
```

## Testing

```bash
just check  # clippy + ruff + pyright + ty + prettier
just test   # cargo nextest + pytest
```

Tests run via `cargo-nextest`. CLI snapshot tests use trycmd — each
`tests/cmd/<name>.toml` defines one CLI invocation. Use `fs.sandbox = true` and
pass `.` as the directory for deterministic output paths.

## Formatting

Always run `just format` before finishing work. This runs `cargo fmt` (Rust),
`ruff format` (Python), and `prettier` (Markdown).

Python targets `>=3.14`, so PEP 758 applies: use the unparenthesized
`except X, Y:` style for multi-exception clauses. This is what `ruff format`
enforces.

## Conventions

- **Unix-only**: uses `std::os::unix::fs::MetadataExt` for ctime/mtime.
- **Explicit deletion**: all foreign keys use `ON DELETE RESTRICT`. Application
  code must delete child rows before parent rows. Never use `ON DELETE CASCADE`.
- **Python dependencies welcome**: `uv` is available on the NAS. Prefer stdlib
  when sufficient, but don't reimplement what a well-maintained package does.
- **StrEnum types**: `EpisodeType`, `BonusType`, `MetadataProvider` in
  `types.py`. Use enum constants (not raw strings) for comparisons.
- **Non-mutating matching**: `_match_files_to_season` returns `MatchedFile`
  wrappers; never mutate `SourceFile.parsed` after initial creation/enrichment.
- **Anime management architecture**: see the "Anime Collection Manager" section
  in `docs/DESIGN_NOTES.md` for data flow, manifest workflow, and special
  episode handling.
- **Media parser**: when working on the parser or investigating QA issues, read
  `docs/etp-anime-parsing-rules.md` — especially the "Non-obvious parsing
  behavior" section which documents edge cases and the reasoning behind them.
  Design decisions for the anime management system are in the "Design decisions"
  section of `docs/DESIGN_NOTES.md`.

## Git workflow

Branch protection is enabled on `main`. All changes go through feature branches
and pull requests.

Large multi-subproject efforts use a long-lived feature branch. Individual
subproject branches merge into the feature branch via PR. Only merge the feature
branch to `main` once the entire effort is complete.

## Documentation

- Implementation plans: `docs/plans/YYYY-MM-DD-plan-name.md`
- Architecture decision records: `docs/adrs/YYYY-MM-DD-NN-decision-name.md`
- Architecture and implementation: `docs/DESIGN_NOTES.md`

Record new architectural decisions as ADRs (Nygard template). When a decision
supersedes an earlier one, update both with cross-references.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/othiym23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
