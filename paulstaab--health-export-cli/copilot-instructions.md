## health-export-cli

> Rust CLI (`health-export-cli`) that parses Apple Health export ZIP files and extracts running workout data. Most logic lives in `src/lib.rs`; `src/main.rs` is a thin entrypoint.

# Agent Instructions

## Project overview

 Rust CLI (`health-export-cli`) that parses Apple Health export ZIP files and extracts running workout data. Most logic lives in `src/lib.rs`; `src/main.rs` is a thin entrypoint.

## CLI structure

```
health-export-cli [--file <PATH>] running <SUBCOMMAND>
  list [--year Y [--month M] | --from D --to D]
                  Print a markdown table of all running workouts (with 1-based # index)
  records [--year Y [--month M] | --from D --to D]
                  Print record rows for longest run and fastest standard distances
  show <N|latest> Show detail view for workout at 1-based index N, or the most recent workout with `latest`
```

`--file` is a root-level option and defaults to `./export.zip`; `--year`, `--month`, `--from`, and `--to` belong to `running list` and `running records`, not `running show`.

## Key data sources

- `export.xml` inside the ZIP — all workout and heart rate data
- `workout-routes/route_*.gpx` files inside the ZIP — GPS trackpoints (lat/lon/time, ~1 Hz)
- Heart rate: `<Record type="HKQuantityTypeIdentifierHeartRate" .../>` elements in `export.xml`; time-windowed, matched against workout/split time ranges by overlap

Workouts have **no unique ID** in the export format. Identification is by 1-based position in the sorted list.

## Code structure (`src/lib.rs`)

| Item | Purpose |
|---|---|
| `Cli / Commands / RunningArgs / RunningSubcommand` | clap 4 derive hierarchy |
| `RunningWorkout` | Parsed workout; `#[derive(Default)]` — use `..Default::default()` in test literals |
| `WorkoutChildData` | Collects child-element data (distance, energy, metadata, GPX path) during XML parsing |
| `GpxPoint` | lat/lon/time from a GPX trackpoint |
| `HrRecord` | Heart rate window (start/end/bpm) |
| `KmSplit` | Per-km split (km number, duration_secs, avg_hr) |
| `extract_running_workouts` | Streaming XML parse; entry point for `list` |
| `read_workout_children` | Reads `<Workout>` child elements; handles `WorkoutStatistics`, `MetadataEntry`, `WorkoutRoute`/`FileReference` |
| `parse_workout_attrs` | Builds `RunningWorkout` from element attributes + child data; uses `attr.unescape_value()` for string fields |
| `parse_gpx` | Streaming GPX parse; extracts `GpxPoint` list |
| `collect_heart_rate` | Second pass through `export.xml`; returns HR records overlapping a time window |
| `compute_splits` | Haversine accumulation over GPX points → `Vec<KmSplit>` |
| `haversine_km` | Inline haversine formula, no extra dependency |
| `print_markdown_table` | Polars DataFrame → markdown table; first column is `#` index |
| `print_show` | Key-value detail + km splits table; omits HR column when no HR data |
| `show_workout` / `compute_workout_splits` | Orchestrate GPX + HR loading for `show`/`latest`; open ZIP in separate scoped blocks |

## Important implementation details

- **XML parsing uses `quick-xml` in non-namespace mode** — element names are plain (`trkpt`, not `{ns}trkpt`).
- **`zip` name conflicts with polars glob import** — always use `::zip::ZipArchive` and `::zip::read::ZipFile<'_>` (crate-root prefix).
- **ZIP opened multiple times for `show`/`latest`**: once for workouts, once for the GPX file, once for HR. Each open is scoped to its own block so borrows don't overlap.
- **`attr.unescape_value()`** must be used (not raw `attr.value`) for string fields that may contain XML entities (e.g. the `device` attribute contains `<` / `>`).
- **GPX times** are RFC 3339 (`2022-10-06T06:01:12Z`); use `DateTime::parse_from_rfc3339`.
- **Health export times** are `"YYYY-MM-DD HH:MM:SS +ZZZZ"`; use `parse_datetime()`.
- **`chrono::DateTime<FixedOffset>` comparisons** are timezone-aware — safe to compare GPX (UTC) against health export (local offset).
- `WorkoutRoute` is a `Start` event (has children), not `Empty`. Use a separate `route_buf` inside the inner loop to avoid borrow conflicts with the outer `buf`.

## Testing

```bash
cargo test                                               # all automated tests must pass
cargo run -- --file example/export.zip running list
cargo run -- --file example/export.zip running show latest
cargo run -- --file example/export.zip running show 1
cargo run -- --file example/export.zip running show 999   # should error
cargo run -- --file example/export.zip running list --year 2024
cargo run -- --file example/export.zip running records --year 2025 --month 2
```

The example ZIP is at `example/export.zip` (157 MB). Example GPX files are under `example/apple_health_export/workout-routes/`.

## Benchmarking

Use `./scripts/benchmark-release.sh` to benchmark the production binary against `example/export.zip` when requested to do so.

The script:

- Builds `target/release/health-export-cli` with `cargo build --release` if the release binary does not exist yet.
- Runs `running list` once for the full export and once per year for 2016 through 2025; the per-year outputs are also used to identify the first workout in each year for the `running show` benchmarks.
- Runs `running show <RUN_ID>` for the first chronologically listed workout in each year from 2016 through 2025.
- Runs `running records` once for the full export and once per year for 2016 through 2025.
- Measures and reports the runtime for every individual command plus a suite total.

Useful variants:

```bash
./scripts/benchmark-release.sh
./scripts/benchmark-release.sh --from-year 2016 --to-year 2016
./scripts/benchmark-release.sh --binary target/release/health-export-cli --export example/export.zip
```

## After every change

Run these three commands in order and fix any issues before considering work done:

```bash
cargo fmt                  # auto-format; never leave unformatted code
cargo clippy               # must produce zero warnings
cargo test                 # all tests must pass
```

`cargo fmt` rewrites the files in place. `cargo clippy` without extra flags uses the default lint set — treat every warning as a required fix, not a suggestion.

Keep `docs/requirements.md` and `docs/test-cases.md` up to date:

- **New or changed behaviour** → add or update requirements in `docs/requirements.md` using the existing ID scheme (`CLI-*`, `PARSE-*`, `COMP-*`, `OUT-*`). Never reuse a retired ID.
- **New, changed, or removed tests** → reflect the change in `docs/test-cases.md`. Mark automated tests with ✅ and the test function name; mark gaps with ❌. Update the `Automated` column whenever a previously manual case gets a test, or vice-versa.

## Coding conventions

- No new crate dependencies without a strong reason — all current needs are met by `clap`, `zip`, `quick-xml`, `polars`, `chrono`, `anyhow`.
- `RunningWorkout` uses `#[derive(Default)]`; test literals should use `..Default::default()` for fields not under test.
- Streaming XML throughout — never load the entire file into memory.

---
> Source: [paulstaab/health-export-cli](https://github.com/paulstaab/health-export-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
